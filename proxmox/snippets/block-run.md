# Block VM/LXC before running

1. Создать сниппет `block_together.pl` с указанием id контейнеров для проверки *@all_vms*
    ```perl
    #!/usr/bin/perl

    use strict;
    use warnings;

    my $vmid = shift;
    my $phase = shift;

    # НАСТРОЙКА: Перечислите через запятую ID всех конфликтующих машин
    my @all_vms = (101, 102, 103);

    # Проверяем стадию перед запуском (pre-start)
    if ($phase eq 'pre-start') {

        # Проверяем, входит ли текущая запускаемая машина в наш список
        if (grep { $_ == $vmid } @all_vms) {

            # Пробегаемся по всем машинам из списка
            foreach my $other_vm (@all_vms) {

                # Пропускаем проверку самой себя
                next if ($other_vm == $vmid);

                # Проверяем статус другой машины
                my $status = `qm status $other_vm 2>/dev/null`;

                if ($status && $status =~ /status: running/) {
                    print "\n[ОШИБКА БЛОКИРОВКИ] Запуск невозможен!\n";
                    print "Виртуальная машина $other_vm сейчас РАБОТАЕТ.\n";
                    print "В данный момент разрешена работа только одной машины из списка (@all_vms).\n\n";
                    exit 1; # Отменяем запуск текущей VM
                }
            }
        }
    }

    exit 0;
    ```
1. Установить права
    ```bash
    chmod +x /var/lib/pve/local-bpc/snippets/block_together.pl
    ```
1. Привязать скрипт к машинам по id
    ```bash
    qm set 101 --hookscript local:snippets/block_together.pl
    qm set 102 --hookscript local:snippets/block_together.pl
    qm set 103 --hookscript local:snippets/block_together.pl
    ```

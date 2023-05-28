# Managing Crons - Linux (Debian)

## Install cron

First thing make sure that `cron` is installed:

```shell
which cron
```

If it's not the case simply install it by executing the following command:

```shell
sudo apt-get install cron
```

## Cron structure

```shell
* * * * * command_to_be_executed
| | | | |
| | | | +----- Day of the Week   (0 - 6) (Sunday = 0)
| | | +------- Month             (1 - 12)
| | +--------- Day of the Month  (1 - 31)
| +----------- Hour              (0 - 23)
+------------- Minute            (0 - 59)
```

# Create a new cron

Let's assume we want to run a `python` script each minute, we have to run the following command to edit the `crontab`:

```shell
crontab -e
```

then add a new line representing the shell command to run, in our case it would be:

```shell
*/1 * * * * python /path_to_script/script.py
```

You can use [https://crontab.guru](https://crontab.guru) to get the right cron to be used.

You can then show the list of crons using 
```shell
crontab -l
```

⚠️ to manage crons using `python`, you can use `python-crontab`
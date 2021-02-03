# Sentinel snippets

## S1 - Script to detect sentinel in "uncertain state"

This script - to be run by crontab - queries sentinel on web port and triggers action if it's not protecting.

Make sure you have web monitor activated and available from localhost.  
You can check via `curl http://localhost:80` , that shoudl return html code of sentinel status web page.


/root/**check_sentinel.py**
```
#!/usr/bin/env python3
from requests import get
from subprocess import check_output

HOST = 'localhost'
PORT = 80
# set WEBHOOK = False for no webhook
WEBHOOK = "https://some_webhook_to_alert"

test = check_output(f'curl -s http://{HOST}:{PORT}/ | sed "s/>/\\n/g" |grep Protecting', shell=True)
test = test.decode().strip()[-5:-2]
if test == '080':
    print("OK")
else:
    print("Sentinel not protecting")
    if WEBHOOK:
        get(WEBHOOK)
    # uncomment and add auto shell action there        
    #check_output("cd /root;./sentinel_reload.sh", shell=True)
```
Then `chmod +x check_sentinel.py`  to have it executable

Add in your crontab, typically:

`*/10 * * * * /root/cron_sentinel.py`

You can uncomment the action line in cron_sentinel.py and have a sentinel_reload.sh that does what auto sync/clear/restart you want it to do.


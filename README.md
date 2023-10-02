Jasmin SMS gateway script for Zabbix
1. Create a script on the zabbix server (the default location for placing the script can be retrieved from zabbix_server.conf inside /etc/zabbix).
```bash
sudo nano /usr/lib/zabbix/alertscripts/jasmin_send.sh
```
2. Copy the following code and paste it inside the jasmin_send.sh file. Make sure to replace the following variables inside the script.
    - jasmin_url
    - jasmin_port
    - sms_shortcode
    - username
    - password
```bash
#!/bin/bash
#
# Default zabbix script path: /usr/lib/zabbix/alertscripts
#
# Defining jasmin server information
jasmin_url="http://jasmin_gatway_ip_or_url"
jasmin_port="jasmin_port"

# Checing jasmin server status
jasmin_status=$(curl -s -X GET $jasmin_url:$jasmin_port/ping)

if [[ $jasmin_status == *"PONG"* ]]; then
    # Retriving parameters
    to=$1
    message="$2 $3"
    sms_shortcode="sms_shortcode_from_your_provider"
    username="jasmin_user"
    password="jasmin_password"
    
    # Log the output of the bash script to a file
    exec 3>&1 4>&2
    trap 'exec 2>&4 1>&3' 0 1 2 3
    exec 1>>/var/run/zabbix/log/jasmin_log.log 2>&1
    
    echo "$(date +%Y-%m-%d_%H:%M:%S): Sending SMS Text to $1"
    
    #Sending the request with all the necessary parameters
    response=$(curl -X POST "$jasmin_url:$jasmin_port/send" \
        --data-urlencode "to=$1" \
        --data-urlencode "from=$sms_shortcode" \
        --data-urlencode "content=$message" \
        --data-urlencode "username=$username" \
    --data-urlencode "password=$password")
    
    # Check the response
    if [ $? -eq 0 ]; then
        if [[ $response == *"Success"* ]]; then
            echo "Message sent successfully!"
        else
            echo "Error sending message: $response"
        fi
    else
        echo "Error making the HTTP request."
    fi
    echo "$(date +%Y-%m-%d_%H:%M:%S): SMS Text sent to $1"
    echo "MESSAGE: $2"
    echo -e "---------------------------------------------------------------------------------------\n"
else
    echo "Error making the HTTP request"
fi
```
3. Give the script the proper permissions and make it executable.
```bash
sudo chown zabbix:zabbix /usr/lib/zabbix/alertscripts/jasmin_send.sh
sudo chmod 777 /usr/lib/zabbix/alertscripts/jasmin_send.sh
```
4. Create the file where the logs of the script will be appended.
```
sudo touch /var/run/zabbix/log/jasmin_log.log
```
5. Go to your Zabbix frontend > Alerts > Media Type and create a new media type.
6. Configure the new media type accordingly.
   - Name: Jasmin
   - Type: Script
   - Script name: jasmin_send.sh
   - Script parameters:
     - {ALERT.SENDTO}
     - {ALERT.SUBJECT}
     - {ALERT.MESSAGE}
   - Enabled: yes
7. Finally, click on Add.

You can test the media type by going to the list of media types and clicking on the test.

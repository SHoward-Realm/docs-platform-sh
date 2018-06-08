# Verify Port Access

Certain factors may impact external \(non-`localhost`\) access to the Realm Object Server’s synchronization facility, and the Realm Dashboard. In order to enable access, it may be necessary to open port 9080 on your server\(s\).

Using the standard Linux tools, these commands will open access to the port:

{% tabs %}
{% tab title="RHEL/CentOS 6" %}
```text
sudo iptables -A INPUT -p tcp -m tcp --dport 9080 -j ACCEPT
sudo service iptables save
```

Please refer to the [CentOS 6  
Documentation](https://wiki.centos.org/HowTos/Network/IPTables) for more information regarding how to configure your firewall.
{% endtab %}

{% tab title="RHEL/CentOS 7" %}
```text
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --add-port=9080/tcp --permanent
sudo firewall-cmd --reload
```
{% endtab %}

{% tab title="Ubuntu 16.04" %}
```text
sudo ufw allow 9080/tcp
sudo ufw reload
```
{% endtab %}
{% endtabs %}

If your environment does not use your distribution's standard firewall, you will have to modify these instructions to fit your environment.



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 


# HowTo get multiple IPV6 addresses with at&t and opnsense

This document is a continuation of a very nice script I originally found for the 'opposing' firewall to opnsense, but had to abandon after it destroyed itself (Opnsense seems to do this less often that the 'other' firewall.)

I found a very nice document [here](https://github.com/lilchancep/att-pfsense-ipv6/blob/1ef29f51708e3fc04a028d62cc714c83c802bca6/README.md)

Short answer:

1. Go to WAN interface: (in interfaces menu)

   1. Ipv6 configuration the: dhcpv6
   2. go to the 'basic' tab and be sure to 'prefix delegation size to 60'
   3. go to 'configuration file override' (you will be creating this later!) choose '/usr/local/etc/rc.d/att-rg-dhcpv6-pd.conf'

   "save"

2. Creating the file: this is a little complex, but short answer is figure out what your interfaces are (wan, lan blah), assign to what is called a "DUID" Say 0 for lan ...). For more exact description, see reference to 'other' firewall document listed above.  Final config should look something like this (DO NOT COPY INTENTIONALLY BROKEN!)

```ini
interface <your interface index eg. igb1 for wan> {
	send ia-na 0;
	send ia-pd 0;
	send ia-pd 1;
	send ia-pd 2;
	<additional PDS for up to 7 networks>
	request domain-name-servers;
	request domain-name;
	script "/var/etc/dhcp6c_wan_script.sh";
};
id-assoc na 0 { };
id-assoc pd 0 {
	prefix-interface <lan pefix here its igb3> {
		sla-id 0;
		sla-len 0;
	};
};
id-assoc pd 1 {
	prefix-interface igb0 {
		sla-id 0;
		sla-len 0;
	};
};
id-assoc pd 2 {
	prefix-interface igb2 {
		sla-id 0;
		sla-len 0;
	};
};
```

I have intentionally 'broken' this configuraiton so you actually have to READ. Please referr to the nice other [document](https://github.com/lilchancep/att-pfsense-ipv6/blob/1ef29f51708e3fc04a028d62cc714c83c802bca6/README.md)

This explains things nicely. Basically each interface gets a 'prefix' delegation which corresponds to each of the 'associated' pd #'s in the configuration file.  Each interface should correspond to appropritate opt1/wan/lan/DMZ whatnot you have defined in your firewall.

## Howto enable SSH in opnsense

In order to create this file, you will need to do three things:

1. Create a new admin user

2. add your ssh public key to enable login

3. make sure the admin use is a member of the 'admins' group

4. enable ssh.

   ------

   Go to 'system', access, users, create a new user who is a member of the admins group

- change default shell to /usr/local/bin/bash
- member of 'admins'
- add 'authorized keys' and paste in your ssh key.

------

Go to 'system, settings, administration'

- go to 'secure shell' section
- enable secure shell
- listen interfaces 'lan'
- key exchange algo: curve25519-sha256
- Ciphers: chacha20-poly1305@openssh.com
- Macs: umac-128-etm@openssh.com
- Host-key-algo: ssh-ed25519

## How to create configuration file

I am NOT going to tell you how to do all of this, only how to get to the 'root' user so you can edit the file. If you can't edit the file OH WELL. I AM NOT GOING TO TRAIN ON VIM etc.

`ssh yourusername@ip.of.your.file` (assuming you ahve already setup ssh for [passwordless authentication](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)!)

you will see the shell prompt

`su -`

you will see the pfsense menu

choose `'exit to shell'`

Finally, cd to the following directory:

`cd /usr/local/etc/rc.d/`

`vi att-rg-dhcpv6-pd.conf`

copy in the text from the config section (be sure to intelligently edit, as it will NOT work w/o thinking!)

Finally, once this is done assign interfaces in the GUI.

## Assigning new interfaces according to the configuration file

Remember that each interface corresponds to the PD # you created in configuration file. You get from 0-7. Others are reserved for at&t use. 

- Go to interface, (select the interface you want to change)
- choose 'track interface' for ipv6 (ipv4 as normal)
- on the 'track ipv6 interface' menu, choose 'wan'
- on ipv6 prefix ID choose the # which corresponds to the PD # in the configuraiton file
- hit 'save'

Many thanks to the orignal author, **ttmcmurry** and others. I just wanted to have a 'opnsense' doc that others can reference to help with this.





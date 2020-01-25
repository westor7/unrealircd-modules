# unrealircd-modules
These are additional modules for [UnrealIRCD](https://www.unrealircd.org/). The modules contained in the directory "unreal5" are developed for the unrealircd-5.0.0 version.

There are also older modules, known to work on the unrealircd-4.2.0 version, in the directory "unreal4". These are unsupported. That means you can try to download and use them, but nobody will help you with these versions.

## Unreal 5.x.x modules

### geoip-base
This one provides data for other "geoip" modules (currently there is only one available, "geoip-whois").

This module needs to be loaded on only single server on the network. (You may keep it active on a second one for redundancy, it won't break anything.)

The module looks for a config block:
```C
geoip {
	ipv4-blocks-file "GeoLite2-Country-Blocks-IPv4.csv";
	ipv6-blocks-file "GeoLite2-Country-Blocks-IPv6.csv";
	countries-file "GeoLite2-Country-Locations-en.csv";
};
```
If one of blocks files is missing, the address type is ignored by the module. If more files can't be loaded, the module fails.

If this config block is not given, it defaults to looking for three files in conf/:
GeoLite2-Country-Blocks-IPv4.csv, GeoLite2-Country-Locations-en.csv, GeoLite2-Country-Blocks-IPv6.csv.
These can be downloaded from [here](https://dev.maxmind.com/geoip/geoip2/geolite2/#Downloads) (get GeoLite2 Country in CSV format). We are notified that the download method may change soon.

### geoip-whois

This one appends swhois info to all users, unless they are not listed in the geoip data.

This module needs to be loaded on only single server on the network (it'll serve the whole network), and requires the "geoip-base" module loaded on same server.

The module looks for a config block:
```C
geoip-whois {
	display-name; // Poland
	display-code; // PL
//	display-continent; // Europe
	info-string "connected from "; // remember the trailing space!
};
```

Display option left out means that this info won't be displayed. (Keep at least one enabled.) No info-string text will cause the module to default to "connected from ".

### geoip-chanban

This one allow banning users from certain countries on a channel. Exceptions and invite exceptions are also possible.

`/mode #channel +b ~C:FR` - will prevent all users from France from joining.

`/mode #channel +iI ~C:RO` - only users from Romania will be able to join.

`/mode #channel +be *4*!*@* ~C:PL` - only users from Poland are allowed to have a number "4" in their nick.

Load this module on every server, together with geoip-base (on two servers for redundancy) or geoip-transfer (on remaining ones).

### geoip-transfer

This one transfers data that come from the geoip-base module loaded on other server, so you don't have to use the resource-intensive geoip-base everywhere. It may be needed by the "geoip-chanban".

### unauthban
This one is created as an attempt of making behaviour of the +R chanmode more selective. It allows things like:

`~I:*!*@*.someisp.com` - lets users from someisp in only when they are registered - this is the particular target
of creating this module.

`~I:~q:~c:#channel` - allows users coming from #channel to talk only when they are registered.

### showwebirc
This one appends swhois info to users that are connected with WEBIRC authorization.

### wwwstats

This one allows the Unreal to cooperate with a web statistics system. Two interfaces are used:

1. UNIX socket. The socket is created on a path specified in config block. When you connect to the socket, the module "spits out" all the current data in JSON format and closes. You can test it with the shell command `socat - UNIX-CONNECT:/tmp/wwwsocket.sock`. It can be used to generate channel lists, server lists, view user counts etc in realtime. Example data:
```json
{
	"clients": 19,
	"channels": 4,
	"operators": 18,
	"servers": 2,
	"messages": 1459,
	"serv": [{
		"name": "test1.example.com",
		"users": 2
	}],
	"chan": [{
		"name": "#help",
		"users": 1,
		"messages": 0
	}, {
		"name": "#services",
		"users": 8,
		"messages": 971
	}, {
		"name": "#opers",
		"users": 1,
		"messages": 0,
		"topic": "This channel has some topic"
	}, {
		"name": "#aszxcvbnm",
		"users": 2,
		"messages": 485
	}]
}
```
2. MySQL database.

Due to incompatibility with the Unreal's module manager, MySQL support is completely disabled by default. Enable it this by changing the line `#undef USE_MYSQL` to `#define USE_MYSQL` in the module source. Please note that, by default, this change will be overriden by the Module Manager when you run "make" (detecting that your version differs from the published one and re-downloading). You can avoid it by temporary commenting out the official module repository in conf/modules.sources.list.

The module periodically inserts new data to the database, unless the data had not changed since the last insert. This can be used to generate graphs, view previous channel topics etc. You should specify database host (localhost is recommended), user, password and database name. Table structure will be created automatically. The structure is:
```sql
CREATE TABLE IF NOT EXISTS `chanlist` (`id` int(11) NOT NULL AUTO_INCREMENT, `date` int(11), `name` char(64), `topic` text, `users` int(11),  `messages` int(11), PRIMARY KEY (`id`), UNIQUE KEY `name` (`name`,`users`,`messages`), KEY `name_3` (`name`), KEY `date` (`date`) )
CREATE TABLE IF NOT EXISTS `stat` (`id` int(11) NOT NULL AUTO_INCREMENT, `date` int(11), `clients` int(11), `servers` int(11), `messages` int(11), `channels` int(11), PRIMARY KEY (`id`), UNIQUE KEY `changes` (`clients`,`servers`,`messages`,`channels`), KEY `date` (`date`) )
```
For obvious reasons you should not enable MySQL on more than one server on your network.

Compiling with mysql support needs mysql client libraries installed on your system. The module is compiled with the command `EXLIBS="-lmysqlclient" make`. If you happen to compile without the EXLIBS option, the module won't load. In such case you should `rm src/modules/third/m_wwwstats.so` and then retry.

+p / +s channels are always ignored.

Message counters are not very precise, as the module counts only messages going through the server it is loaded on. That means that some channels at some time can not be counted.

The module looks for a config block:
```C
wwwstats {
	socket-path "/tmp/wwwstats.sock";	// do not specify if you don't want the socket
	use-mysql;	// remove this line if you don't want mysql
	mysql-interval "900"; // time in seconds, default is 900 (15 minutes)
	mysql-host "localhost";
	mysql-db "database";
	mysql-user "username";
	mysql-pass "password";
};
```
### findchmodes

This one allows IRCoperators to check which channels use certain channel mode. You can use it to check, for example, who has the Channel History enabled.

Usage example:

`/findchmodes +H`

## Unreal 4.x.x modules

Remember that modules listed below are now unsupported.

### m_geoip_whois
This one appends swhois info to all users, unless they are not listed in the input data.

This module needs to be loaded on only single server on the network.

The module looks for a config block:
```C
geoip-whois {
	ipv4-blocks-file "GeoLite2-Country-Blocks-IPv4.csv";
	ipv6-blocks-file "GeoLite2-Country-Blocks-IPv6.csv";
	countries-file "GeoLite2-Country-Locations-en.csv";
	display-name; // Poland
	display-code; // PL
//	display-continent; // Europe
	info-string "connected from "; // remember the trailing space!
};
```

If one of blocks files is missing, the address type is ignored by the module. If more files can't be loaded, the module fails. Display option left out means that this info won't be displayed. (Keep at least one enabled.) No info-string text will cause the module to default to "connected from ".

If this block is not given, it works like the old version, defaulting to the options in example above, and looking for three files in conf/:
GeoLite2-Country-Blocks-IPv4.csv, GeoLite2-Country-Locations-en.csv, GeoLite2-Country-Blocks-IPv6.csv.
These can be downloaded from [here](https://dev.maxmind.com/geoip/geoip2/geolite2/#Downloads) (get GeoLite2 Country in CSV format).

### m_unauthban
This one is created as an attempt of making behaviour of the +R chanmode more selective. It allows things like:

`~I:*!*@*.someisp.com` - lets users from someisp in only when they are registered - this is the particular target
of creating this module.

`~I:~q:~c:#channel` - allows users coming from #channel to talk only when they are registered.

### m_showwebirc
This one appends swhois info to users that are connected with WEBIRC authorization.

### m_wwwstats

This one allows the Unreal to cooperate with a web statistics system. Two interfaces are used:

1. UNIX socket. The socket is created on a path specified in config block. When you connect to the socket, the module "spits out" all the current data in JSON format and closes. You can test it with the shell command `socat - UNIX-CONNECT:/tmp/wwwsocket.sock`. It can be used to generate channel lists, server lists, view user counts etc in realtime. Example data:
```json
{
	"clients": 19,
	"channels": 4,
	"operators": 18,
	"servers": 2,
	"messages": 1459,
	"serv": [{
		"name": "test1.example.com",
		"users": 2
	}],
	"chan": [{
		"name": "#help",
		"users": 1,
		"messages": 0
	}, {
		"name": "#services",
		"users": 8,
		"messages": 971
	}, {
		"name": "#opers",
		"users": 1,
		"messages": 0,
		"topic": "This channel has some topic"
	}, {
		"name": "#aszxcvbnm",
		"users": 2,
		"messages": 485
	}]
}
```
2. MySQL database. The module periodically inserts new data to the database, unless the data had not changed since the last insert. This can be used to generate graphs, view previous channel topics etc. You should specify database host (localhost is recommended), user, password and database name. Table structure will be created automatically. The structure is:
```sql
CREATE TABLE IF NOT EXISTS `chanlist` (`id` int(11) NOT NULL AUTO_INCREMENT, `date` int(11), `name` char(64), `topic` text, `users` int(11),  `messages` int(11), PRIMARY KEY (`id`), UNIQUE KEY `name` (`name`,`users`,`messages`), KEY `name_3` (`name`), KEY `date` (`date`) )
CREATE TABLE IF NOT EXISTS `stat` (`id` int(11) NOT NULL AUTO_INCREMENT, `date` int(11), `clients` int(11), `servers` int(11), `messages` int(11), `channels` int(11), PRIMARY KEY (`id`), UNIQUE KEY `changes` (`clients`,`servers`,`messages`,`channels`), KEY `date` (`date`) )
```
For obvious reasons you should not enable MySQL on more than one server on your network.

Compiling with mysql support needs mysql client libraries installed on your system. The module is compiled with the command `EXLIBS="-lmysqlclient" make`. If you happen to compile without the EXLIBS option, the module won't load. In such case you should `rm src/modules/third/m_wwwstats.so` and then retry.

Optionally, MySQL support can be completely removed (when you don't want to use it, and don't have the client libraries installed). Do this by changing the line `#define USE_MYSQL` to `#undef USE_MYSQL` in the module source.

+p / +s channels are always ignored.

Message counters are not very precise, as the module counts only messages going through the server it is loaded on. That means that some channels at some time can not be counted.

The module looks for a config block:
```C
wwwstats {
	socket-path "/tmp/wwwstats.sock";	// do not specify if you don't want the socket
	use-mysql;	// remove this line if you don't want mysql
	mysql-host "localhost";
	mysql-db "database";
	mysql-user "username";
	mysql-pass "password";
};
```

## Licensing information

You can use, modify and share the modules according to the [GPLv3](https://www.gnu.org/licenses/gpl-3.0.html) license, unless it's stated differently inside the source code file.

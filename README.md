<div align="center">

## IRC Bot


</div>

### Description

This is an standalone IRC Bot written in PHP (no IRC client needed) It's written as a generic class so that you can derive from it and create your own personal bot.
 
### More Info
 
You should at least know how to use IRC. Full understanding of the protocol is not necessary.

Don't use this as a web php application. Use it at command line.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[hobbit125](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/hobbit125.md)
**Level**          |Intermediate
**User Rating**    |4.7 (14 globes from 3 users)
**Compatibility**  |PHP 4\.0
**Category**       |[Complete Applications](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/complete-applications__8-7.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/hobbit125-irc-bot__8-485/archive/master.zip)

### API Declarations

```
Do as you will with this. If you have any questions email me hobbit125@summerof98.com
I'd appreciate it if you gave me credit when using this.
```


### Source Code

```
THIS FILE SHOULD BE NAMED (ircBot.php)
<?
    // this is the base class. do NOT
    // instatiate this. you need to write
    // a class to derive from it and implement
    // a constructor and override all of the
    // on_ functions. An example of a derived
    // runnable bot is listed beneath.
	class IRC_Bot {
  	var $nick;
  	var $username;
		var $description;
		var $localhost;
		var $remotehost;
		var $remoteport;
		var $echoincoming;
		var $ircsocket;
		function IRC_Bot() {
	    set_time_limit(0);
			ob_end_flush();
			echo "\r\n";
		}
		function bot_connect () {
			// connect to IRC server
			$this->ircsocket = fsockopen ($this->remotehost, $this->remoteport) ;
			if (! $this->ircsocket) {
				die ("Error connecting to host.");
			}
			print "Connected to: $this->remotehost:$this->remoteport\n";
			fputs ($this->ircsocket, "USER $this->username $this->localhost $this->remotehost: $this->description\r\n");
			fputs ($this->ircsocket, "NICK $this->nick\r\n");
		}
		function bot_go () {
			// IRC loop
			while (!feof($this->ircsocket)) {
				$incoming = fgets ($this->ircsocket, 1024);
				$incoming = str_replace( "\r", "", $incoming);
				$incoming = str_replace("\n", "", $incoming);
				if ($this->echoincoming) echo $incoming . "\n";
				if (substr($incoming, 0, 1) == ":") {
					$prefix = substr ($incoming, 0, strpos($incoming, ' '));
					$incoming = substr ($incoming, strpos($incoming, ' ') + 1);
				} else {
					$prefix = "";
				}
				$command = substr ($incoming, 0, strpos($incoming, ' '));
				$incoming = substr ($incoming, strpos($incoming, ' ') + 1);
				$params = explode (" ", $incoming);
				if ($command == "PING") fputs($this->ircsocket, "PONG\r\n");
				$this->bot_parse ($prefix, $command, $params);
			}
			fputs($this->ircsocket, "QUIT Unexpected\r\n");
		}
		function bot_parse ($prefix, $command, $params) {
			if ($command == "PRIVMSG") {
				$nick = substr ($prefix, strpos($prefix, ":") + 1, strpos($prefix, "!") - 1);
				$ident = substr ($prefix, strpos($prefix, "!"));
				$target = array_shift ($params);
				$params[0] = substr ($params[0], 1);
				if (substr($target, 0, 1) == "#") {
					$this->on_channel_msg ($nick, $ident, $target, $params);
				} else {
					$this->on_private_msg ($nick, $ident, $params);
				}
			}
			if ($command == "NOTICE") {
				$nick = substr ($prefix, strpos($prefix, ":") + 1, strpos($prefix, "!") - 1);
				$ident = substr ($prefix, strpos($prefix, "!"));
				array_shift ($params);
				$params[0] = substr ($params[0], 1);
				$this->on_notice ($nick, $ident, $params);
			}
		}
		////////////////////////////////////////////////////
		//				IRC FUNCTIONS (call these to perform various irc tasks.)	 //
		////////////////////////////////////////////////////
		function irc_write ($message) {
			fputs ($this->ircsocket, $message . "\r\n");
		}
		function irc_join ($channel) {
			$this->irc_write("JOIN $channel");
		}
		function irc_part($channel) {
			$this->irc_write("PART $channel");
		}
		function irc_quit ($reason) {
			$this->irc_write("QUIT :$reason");
		}
		function irc_notice ($user, $message) {
			$this->irc_write("NOTICE :$message");
		}
		function irc_msg ($user, $message) {
			$this->irc_write("PRIVMSG $user :$message");
		}
		function irc_action ($user, $message) {
			$this->irc_write ("PRIVMSG $user :" . chr(1) ."ACTION $message");
		}
		function irc_mode ($channel, $user, $mode) {
			$this->irc_write ("MODE $channel $mode $user");
		}
		function irc_op ($channel, $user) {
			$this->irc_mode ($channel, $user, "+o");
		}
		function irc_deop ($channel, $user) {
			$this->irc_mode ($channel, $user, "-o");
		}
		////////////////////////////////////////////////////
		//				IRC EVENTS (override these in your derived class.)				 //
		////////////////////////////////////////////////////
		function on_private_msg ($nick, $ident, $params) {
		}
		function on_channel_msg ($nick, $ident, $chan, $params) {
		}
		function on_notice ($nick, $ident, $params) {
		}
	}
 ?>
PUT ALL THIS IN A DIFFERENT FILE (silverbot.php):
<?
    // this is an example of a runnable
    // derived bot. run this file at the
    // commandline. DO NOT run it as a
    // web document. it will hang in memory
    // you have been warned.
	define ("BOT_PASSWORD", "fruitloops");
	include ("ircbot.php");
	class Silver_Bot extends IRC_Bot {
		function Silver_Bot ($n = "HBSilver", $r = "irc.dal.net", $p = 6667, $e = true, $d = "Hobbit Bot Silver", $u = "HBSilver", $l = "localhost") {
			$this->IRC_Bot();
			$this->nick = $n;
			$this->username = $u;
			$this->description = $d;
			$this->localhost = $l;
			$this->remotehost = $r;
			$this->remoteport = $p;
			$this->echoincoming = $e;
		}
		function on_notice ($nick, $ident, $params) {
			$password = array_shift ($params);
			if ($password == BOT_PASSWORD) {
				$command = array_shift ($params);
				switch ($command) {
				case "JOIN":
					$this->irc_join ($params[0]);
					break;
				case "PART":
					$this->irc_part ($params[0]);
					break;
				case "QUIT":
					$this->irc_quit (join($params, " "));
					break;
				case "MSG":
					$user = array_shift ($params);
					$this->irc_msg($user, join($params, " "));
					break;
				case "OP":
					$this->irc_op($params[0], $params[1]);
					break;
				case "DEOP":
					$this->irc_deop($params[0], $params[1]);
					break;
				case "ACTION":
					$user = array_shift ($params);
					$this->irc_action($user, join($params, " "));
					break;
				}
			} else {
				$this->irc_msg($nick, "You are not my master.");
			}
		}
	}
	// this instantiates a new silverbot
    // and gets it going.
	$mysilver = new Silver_Bot("HBSilver", "irc.dal.net");
	echo $mysilver->bot_connect();
	echo $mysilver->bot_go();
?>
```


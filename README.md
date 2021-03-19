# Linux Kernel Programming Part 2 - Char Device Drivers and Kernel Synchronization

<a href="https://www.packtpub.com/product/linux-kernel-programming-part-2-char-device-drivers-and-kernel-synchronization/9781801079518"><img src="https://static.packt-cdn.com/products/9781801079518/cover/smaller" alt="Linux Kernel Programming Part 2 - Char Device Drivers and Kernel Synchronization" height="256px" align="right"></a>

This is the code repository for [Linux Kernel Programming Part 2 - Char Device Drivers and Kernel Synchronization](https://www.packtpub.com/product/linux-kernel-programming-part-2-char-device-drivers-and-kernel-synchronization/9781801079518), published by Packt.

**Linux Kernel Programming Part 2 - Char Device Drivers and Kernel Synchronization, published by Packt**

## What is this book about?


This book covers the following exciting features:
* Get to grips with the basics of the modern Linux Device Model (LDM)
* Write a simple yet complete misc class character device driver
* Perform user-kernel interfacing using popular methods
* Understand and handle hardware interrupts confidently
* Perform I/O on peripheral hardware chip memory
* Explore kernel APIs to work with delays, timers, kthreads, and workqueues
* Understand kernel concurrency issues
* Work with key kernel synchronization primitives and discover how to detect and avoid deadlock

If you feel this book is for you, get your [copy](https://www.amazon.com/dp/) today!

<a href="https://www.packtpub.com/?utm_source=github&utm_medium=banner&utm_campaign=GitHubBanner"><img src="https://raw.githubusercontent.com/PacktPublishing/GitHub/master/GitHub.png" 
alt="https://www.packtpub.com/" border="5" /></a>

## Instructions and Navigations
All of the code is organized into folders. For example, Chapter02.

The code will look like the following:
```
static int __init miscdrv_init(void)
{
    int ret;
    struct device *dev;
```

**Following is what you need for this book:**
An understanding of the topics covered in the Linux Kernel Programming book is highly recommended to make the most of this book. This book is for Linux programmers beginning to find their way with device driver development. Linux device driver developers looking to overcome frequent and common kernel/driver development issues, as well as perform common driver tasks such as user-kernel interfaces, performing peripheral I/O, handling hardware interrupts, and dealing with concurrency will benefit from this book. A basic understanding of Linux kernel internals (and common APIs), kernel module development, and C programming is required.

With the following software and hardware list you can run all code files present in the book (Chapter 1-7).
### Software and Hardware List
| Chapter | Software required | OS required |
| -------- | ------------------------------------ | ----------------------------------- |
| 1-7 | Ubuntu 18.04 LTS | Windows, Mac OS X, and Linux (Any) |
| 1-7 | Oracle VirtualBox 6.x | Windows, Mac OS X, and Linux (Any) |

We also provide a PDF file that has color images of the screenshots/diagrams used in this book. [Click here to download it](http://www.packtpub.com/sites/default/files/downloads/9781801079518_ColorImages.pdf).

### Related products
* Mastering Linux Device Driver Development [[Packt]](https://www.packtpub.com/product/mastering-linux-device-driver-development/9781789342048?utm_source=github&utm_medium=repository&utm_campaign=9781789342048) [[Amazon]](https://www.amazon.com/dp/178934204X)

* Linux Kernel Programming [[Packt]](https://www.packtpub.com/product/linux-kernel-development-cookbook/9781789953435?utm_source=github&utm_medium=repository&utm_campaign=9781789953435) [[Amazon]](https://www.amazon.com/dp/178995343X)

## Get to Know the Author
**Kaiwan N Billimoria**
taught himself BASIC programming on his dad's IBM PC back in 1983. He was programming in C and Assembly on DOS until he discovered the joys of Unix, and by around 1997, Linux!

Kaiwan has worked on many aspects of the Linux system programming stack, including Bash scripting, system programming in C, kernel internals, device drivers, and embedded Linux work. He has actively worked on several commercial/FOSS projects. His contributions include drivers to the mainline Linux OS and many smaller projects hosted on GitHub. His Linux passion feeds well into his passion for teaching these topics to engineers, which he has done for well over two decades now. He's also the author of Hands-On System Programming with Linux. It doesn't hurt that he is a recreational ultrarunner too.

## Other books by the authors
* [Hands-On System Programming with Linux](https://www.packtpub.com/networking-and-servers/hands-system-programming-linux?utm_source=github&utm_medium=repository&utm_campaign=9781788998475)



# whats-a-netcat

Point: 100

## Category

General Skills

## Question

Using netcat \(nc\) is going to be pretty important. Can you connect to `2019shell1.picoctf.com` at port `21865` to get the flag?

## Hint

nc [tutorial](https://linux.die.net/man/1/nc)

## Solution

Using netcat to listen on server `2019shell1.picoctf.com` with port `21864`

```text
root@kali:~$ nc 2019shell1.picoctf.com 21864
```

## Flag

picoCTF{nEtCat\_Mast3ry\_4fefb685}


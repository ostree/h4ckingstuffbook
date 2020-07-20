# Mr-Worldwide

Point: 200

## Category

Cryptography

## Question

A musician left us a [message](https://2019shell1.picoctf.com/static/46e165b0a953075440f3a544fdb4cff1/message.txt). What's it mean?

## Hint

NO HINT

## Solution

Look at the text file, I've got this. It seems like coordinates. So, I decided to file these coordinates on Google Map to file the location.

```bash
picoCTF{(35.028309, 135.753082)(46.469391, 30.740883)(39.758949, -84.191605)(41.015137, 28.979530)(24.466667, 54.366669)(3.140853, 101.693207)_(9.005401, 38.763611)(-3.989038, -79.203560)(52.377956, 4.897070)(41.085651, -73.858467)(57.790001, -152.407227)(31.205753, 29.924526)}
```

| Location | Coordinates |
| :--- | :--- |
| Kyoto | 35.028309, 135.753082 |
| Odesa | 46.469391, 30.740883 |
| Dayton | 39.758949, -84.191605 |
| Istanbul | 41.015137, 28.979530 |
| Abu Dhabi | 24.466667, 54.366669 |
| Kuala Lumpur | 3.140853, 101.693207 |
| Addis Ababa | 9.005401, 38.763611 |
| Loja | -3.989038, -79.203560 |
| Amsterdam | 52.377956, 4.897070 |
| Sleepy Hollow | 41.085651, -73.858467 |
| Kodiak | 57.790001, -152.407227 |
| Alexandria | 31.205753, 29.924526 |

It seems that nothing's useful, but then I tried to get the first character of each location and join them together. Yeah!!! That's it. I've got the flag

## Flag

picoCTF{KODIAK\_ALASKA}


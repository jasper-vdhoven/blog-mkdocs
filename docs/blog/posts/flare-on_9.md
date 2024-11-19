---
title: Flare-on 9 CTF
date:
  created: 2022-11-15
description: My attempt at this year's Flare-on challenge
---

# Flare-on 9

My endeavours into playing Flare-on for the first time. Write-up goes up to challeng 8 - Backdoor and covers the process from description to flag.

<!-- more -->

## Introduction

The best and worst events in life are often the ones you didn't plan for or went into unprepared for. This about sums up my first experience with Flare-on and actual reverse engineering in general.

Going into this years Flare-on I didn't have any actual reversing knowledge, having touched Ghidra maybe a handful of times and having toyed with some basic binaries for OSCP's buffer overflow. Now, with Flare-on behind me, I'm looking forward to next year's Flare-on and finishing it in its entirety, having achieved exponentially more than I could have ever dreamed during this year's edition!

First and foremost, a huge thanks to Yassir a.k.a. [@kladblokje88][kladblokje-twitter], and an unnamed friend which will go by his username mryoranimo with whom I participated in these challenges with. mryoranimo is a lot more experienced in reversing than me and helped explain things when my peanut brain couldn't comprehend something. I probably wouldn't have made it this far without you, so a big thank you for that <3.

With the life story covered, let's dig into this years Flare-on!

## Challenge 1: Flaredle

<figure markdown="span">
	![Flare-on 9 Flaredle challenge description](./images/flareon9/flareon9-flaredle-challenge-description.webp)
</figure>

Upon clicking the play live link the website shows a Wordle clone as the challenge description hinted at. Unlike the official Wordle, this clone asks for _a lot_ more characters for each guess. This is probably where the "is too hard to beat without cheating" part of the challenge description comes into play:

<figure markdown="span">
	![Flare-on 9 Flaredle with two guesses being shown](./images/flareon9/flareon9-flaredle-overview.webp)
</figure>

After entering some guesses it appears to act just like the original Wordle game, but due the increased word length it will much more difficult to brute-force shared letter combinations among English words. Onto the source code we go instead.

Looking in `script.js` the following section of code checks whether we have entered the desired word:

```js title="Flaredle JavaScript validation logic" linenums="1"
if (guessString === rightGuessString) {
  let flag = rightGuessString + "@flare-on.com";
  toastr.options.timeOut = 0;
  toastr.options.onclick = function () {
    alert(flag);
  };
  toastr.success("You guessed right! The flag is " + flag);

  guessesRemaining = 0;
  return;
} else {
  guessesRemaining -= 1;
  currentGuess = [];
  nextLetter = 0;

  if (guessesRemaining === 0) {
    toastr.error("You've run out of guesses! Game over!");
    toastr.info(
      'Try reverse engineering the code to discover the correct "word"!'
    );
  }
}
```

Checking the definition of `rightGuessString` shows the following:

```js
let rightGuessString = WORDS[CORRECT_GUESS];
```

With `CORRECT_GUESS` defined as:

```js
const CORRECT_GUESS = 57;
```

Checking the definition of `WORDS` shows that the `words.js` file also provides this:

```js
import { WORDS } from "./words.js";
```

Thus, checking the `words.js` file at line the 57th entry shows the correct and single word containing "flare":

```js
export const WORDS = ['acetylphenylhydrazine',
    [snip]
    'establishmentarianism',
    'flareonisallaboutcats',
    'gastroenterocolostomy',
    [snip]
    'zygomaticoauricularis',
	]
```

Entering the word `flareonisallaboutcats` shows that this is the correct answer and returns the flag:

<figure markdown="span">
	![Flare-on 9 Flaredle flag returned by correct answer](./images/flareon9/flareon9-flaredle-wordle-flag.webp)
</figure>

## Challenge 2: Pixel Poker

<figure markdown="span">
	![Flare-on 9 Pixel Poker description](./images/flareon9/flareon9-pixel-poker-challenge-description.webp)
</figure>

This challenge consisted of two files: an executable named `PixelPoker.exe` and a text file named `readme.txt` with the following in it:

!!! quote
    Welcome to PixelPoker ^\_^, the pixel game that's sweeping the nation!

    Your goal is simple: find the correct pixel and click it
    
    Good luck!

Opening the provided `PixelPoker.exe` shows an application window with a random array of colours with the challenge being to click the correct one. Clicking around to see what happens the following pop-up got displayed, informing that one needs to try again:

<figure markdown="span">
	![Flare-on 9 Pixel Poker Womp Womp please play again](./images/flareon9/flareon9-pixel-poker-womp-womp.webp)
</figure>

Seeing as the last challenge took inspiration from the popular Wordle game, perhaps this took inspiration from [elsewhere][yt-shylilly] too 😉.

Jokes aside this challenge proved _much_ simpler than initially thought, not requiring tools such as IDA, Ghidra, et al to solve. This "easier than expected" outcome was something that continued throughout Flare-on as I started understanding the hints in the challenge name.

Opening the `PixelPoker.exe` binary with 7-zip showed a series of directories, including one containing two bitmaps:

<figure markdown="span">
	![Flare-on 9 Pixel Poker 7-zip showing two bitmaps](./images/flareon9/flareon9-pixel-poker-binary-bitmaps.webp)
</figure>

mryoranimo found some code within the decompiled source in Ghidra doing XOR operations on _something_ we did not quite get just yet. Later, Yassir compared the two images on [diffchecker.com][diffchecker] and noticed the flag at the bottom of the image:

<figure markdown="span">
	![Flare-on 9 Pixel Poker diffchecker.com with flag showing](./images/flareon9/flareon9-pixel-poker-bitmaps-flag.webp)
</figure>

The almost illegible text at the bottom reads `w1nN3r_W!NneR_cHick3n_d1nNer@flare-on.com`, the flag for this challenge.

## Challenge 3: Magic 8 Ball

<figure markdown="span">
	![Flare-on 9 Magic 8 Ball challenge description](./images/flareon9/flareon9-magic8ball-challenge-description.webp)
</figure>

Executing the challenge binary `Magic8Ball.exe` shows a blue application window, within it a Magic 8 Ball one can ask questions:

<figure markdown="span">
	![Flare-on 9 Magic 8 Ball challenge binary](./images/flareon9/flareon9-magic8ball-binary.webp)
</figure>

Running the Mandiant [FLOSS][gh-floss] tool over the binary gives back an interesting strings spelling out `gimme flag pls`:

<figure markdown="span">
	![Flare-on 9 Magic 8 Ball FLOSS output](./images/flareon9/flareon9-magic8ball-floss-output.webp)
</figure>

Messing around a bit in IDA, trying to figure out how it even all worked, I found a series of if-else statements maybe checking for letters such as `L`, `R`, `U`, and `D`:

<figure markdown="span">
	![Flare-on 9 Magic 8 Ball IDA showing checking conditions with character codes](./images/flareon9/flareon9-magic8ball-IDA-characters.webp)
</figure>

Ghidra decompiled this function at address `0x4024e0` to the following C++ code:

```cpp linenums="1" title="Ghidra decompile of character input checking"
    //snip
    if (*(char *)((int)param_1 + 0x159) != '\0') {
    uVar1 = *(uint *)((int)param_1 + 0x124);
    ppcVar4 = this;
    if (0xf < uVar1) {
      ppcVar4 = (char **)*this;
    }
    if (*(char *)ppcVar4 == 'L') {
      ppcVar4 = this;
      if (0xf < uVar1) {
        ppcVar4 = (char **)*this;
      }
      if (*(char *)((int)ppcVar4 + 1) == 'L') {
        ppcVar4 = this;
        if (0xf < uVar1) {
          ppcVar4 = (char **)*this;
        }
        if (*(char *)((int)ppcVar4 + 2) == 'U') {
          ppcVar4 = this;
          if (0xf < uVar1) {
            ppcVar4 = (char **)*this;
          }
          if (*(char *)((int)ppcVar4 + 3) == 'R') {
            ppcVar4 = this;
            if (0xf < uVar1) {
              ppcVar4 = (char **)*this;
            }
            if (*(char *)(ppcVar4 + 1) == 'U') {
              ppcVar4 = this;
              if (0xf < uVar1) {
                ppcVar4 = (char **)*this;
              }
              if (*(char *)((int)ppcVar4 + 5) == 'L') {
                ppcVar4 = this;
                if (0xf < uVar1) {
                  ppcVar4 = (char **)*this;
                }
                if (*(char *)((int)ppcVar4 + 6) == 'D') {
                  ppcVar4 = this;
                  if (0xf < uVar1) {
                    ppcVar4 = (char **)*this;
                  }
                  if (*(char *)((int)ppcVar4 + 7) == 'U') {
                    ppcVar4 = this;
                    if (0xf < uVar1) {
                      ppcVar4 = (char **)*this;
                    }
                    if (*(char *)(ppcVar4 + 2) == 'L') {
                      _Str1 = (undefined4 *)((int)param_1 + 0xf8);
                      if (0xf < *(uint *)((int)param_1 + 0x10c)) {
                        _Str1 = (undefined4 *)*_Str1;
                      }
                      iVar2 = strncmp((char *)_Str1,(char *)((int)param_1 + 0x5c),0xf);
                      if (iVar2 == 0) {
                        FUN_00401220(&stack0xffffffc0,this);
                        FUN_00401a10(param_1,in_stack_ffffffc0);
                      }
                      // snip
```

Putting the entire chain into order shows the following pattern: `L L U R U L D U L`.

mryoranimo and I tried to enter the pattern of arrow-keys in combination with "gimme flag pls?" to no avail. The next day doing it all again it somehow worked this time, ain't that fun 🫠.

Doing the shuffle with the pattern, entering the phrase `gimme flag pls?`, and pressing enter ~~finally~~ returned the flag:

<figure markdown="span">
	![Flare-on 9 Magic 8 Ball flag returned by challenge](./images/flareon9/flareon9-magic8ball-flag.webp)
</figure>

## Challenge 4: darn_mice

<figure markdown="span">
	![Flare-on 9 darn_mice challenge description](./images/flareon9/flareon9-darnmice-challenge-description.webp)
</figure>

Executing the binary without any arguments does nothing and just exists the program afterwards it seems:

<figure markdown="span">
	![Flare-on 9 darn_mice binary output with no input](./images/flareon9/flareon9-darnmice-binary-noinput.webp)
</figure>

Entering a random input as argument returns an output stating "On your plate, you see four olives. You leave the room, and a mouse EATS one!"

<figure markdown="span">
	![Flare-on 9 darn_mice output with random input](./images/flareon9/flareon9-darnmice-binary-random-input.webp)
</figure>

However, giving an input of 36 characters or greater will show a different input: "On your plate, you see four olives. No, nevermind"

<figure markdown="span">
	![Flare-on 9 darn_mice 36 character input](./images/flareon9/flareon9-darnmice-binary-36-character-input.webp)
</figure>

I did notice some strange behaviour when solely entering the letter `s`. The binary returned the following new strings `nibble...`, `When you return, you only: s` followed by a random string of characters:

<figure markdown="span">
	![Flare-on 9 darn_mice binary output with 's' as input](./images/flareon9/flareon9-darnmice-s-input.webp)
</figure>

This "strange behaviour" that Yassir and I had discovered when entering the letter `s` turned out to be part of getting to the flag for this challenge. What we did not yet fully grasp was the *why* for why this specific character returned this gibberish of characters. What Yassir and I had manually brute-forced was the first of 36 bytes that needed to be looped through to get to the challenge flag. This array of 36 bytes is located at offset `0x40100` and is used in a function which takes the value at the current offset, adds our input to it, and then executes it as if it were a valid instruction. The decompiled and annotated code produced by Ghidra can are down below:

```cpp linenums="1" title="Ghidra decompile"
void __cdecl FUN_00401000(PUCHAR param_1)
// snip

// array of 36 bytes
  byte_array[0] = 'P';
  byte_array[1] = 0x5e;
  byte_array[2] = 0x5e;
  byte_array[3] = 0xa3;
  byte_array[4] = 0x4f;
  byte_array[5] = 0x5b;
  byte_array[6] = 0x51;
  byte_array[7] = 0x5e;
  byte_array[8] = 0x5e;
  byte_array[9] = 0x97;
  byte_array[10] = 0xa3;
  byte_array[11] = 0x80;
  byte_array[12] = 0x90;
  byte_array[13] = 0xa3;
  byte_array[14] = 0x80;
  byte_array[15] = 0x90;
  byte_array[16] = 0xa3;
  byte_array[17] = 0x80;
  byte_array[18] = 0x90;
  byte_array[19] = 0xa3;
  byte_array[20] = 0x80;
  byte_array[21] = 0x90;
  byte_array[22] = 0xa3;
  byte_array[23] = 0x80;
  byte_array[24] = 0x90;
  byte_array[25] = 0xa3;
  byte_array[26] = 0x80;
  byte_array[27] = 0x90;
  byte_array[28] = 0xa3;
  byte_array[29] = 0x80;
  byte_array[30] = 0x90;
  byte_array[31] = 0xa2;
  byte_array[32] = 0xa3;
  byte_array[33] = 0x6b;
  byte_array[34] = 0x7f;
  byte_array[35] = 0;

  // snip
  sVar1 = _strlen((char *)param_1);
  uVar3 = SUB41(pcVar4,0);
  if ((sVar1 == 0) || (0x23 < sVar1)) {
    FUN_00401240((wchar_t *)s_No,_nevermind._0041905c);
    uVar2 = extraout_DL;
  }
  else {
    FUN_00401240((wchar_t *)s_You_leave_the_room,_and_a_mouse_E_0041906c);
    for (local_30 = 0;
        ((uVar3 = SUB41(pcVar4,0), local_30 < 0x24 && (byte_array[local_30] != '\0')) &&
        (param_1[local_30] != '\0')); local_30 = local_30 + 1) {
      pcVar4 = (code *)VirtualAlloc((LPVOID)0x0,0x1000,0x3000,0x40);
      *pcVar4 = (code)(byte_array[local_30] + param_1[local_30]);
      (*pcVar4)();
      FUN_00401240((wchar_t *)s_Nibble..._00419098);
    }
    FUN_00401240((wchar_t *)s_When_you_return,_you_only:_%s_004190a4);
    FUN_00401280((int)&DAT_00419000,DAT_00419030,param_1,(PUCHAR)s_salty_004190c4,(int)&DAT_00419000
                 ,DAT_00419030);
    FUN_00401240((wchar_t *)&DAT_004190cc);
    uVar2 = extraout_DL_00;
  }
```

!!! note
    I thought I had made a screenshot of the cleaned up function, but apparently I don't have my Ghidra project nor any screenshots 🥲

The significance of the letter `s` in this function now became clear to us. The letter S is equivalent to `0xC3` or 195, which is the same as the OP code `RET` or return:

| Opcode | Mnemonic | Description                      |
| ------ | -------- | -------------------------------- |
| C3     | RET      | Near return to calling procedure |
| CB     | RET      | Far return to calling procedure  |

Using the documentation under [c9x.me][x86-ret] the `RET` operation is described as the following:

!!! quote
    Transfers program control to a return address located on the top of the stack. The address is usually placed on the stack by a CALL instruction, and the return is made to the instruction that follows the CALL instruction.


The `RET` OP code doesn't cause any exceptions or errors as it returns the program flow to the calling procedure. Different OP codes such as `ADD` or `INC` change the state and would cause an exception, thus exiting the program. Now, with the behaviour of the character `s` clarified the challenge description "If it crashes its user error" and the binary output "when you return you only:" now made sense to me. To ensure the program can loop through all 36 bytes in its array, we need to have it execute the `RET` instruction for each of the 26 steps.

To achieve the above, I used a Cyberchef recipe that took a string of C3s and subtracted that with each item in the array to get the required value for each entry. The Cyberchef recipe and it's output can be seen down below:

<figure markdown="span">
	![Flare-on 9 darn_mice Cyberchef recipe](./images/flareon9/flareon9-darnmice-cyberchef-formula.webp)
</figure>

With this recipe the magic string we need to enter to have everything return `C3` / `RET` is: `see three, C3 C3 C3 C3 C3 C3 C3! XD`. Entering this into the challenge binary spits out a fitting flag of `i_w0uld_l1k3_to_RETurn_this_joke@flare-on.com`:

<figure markdown="span">
	![Flare-on 9 darn_mice flag](./images/flareon9/flareon9-darnmice-flag-output.webp)
</figure>

## Challenge 5: T8

<figure markdown="span">
	![Flare-on 9 T8 challenge description](./images/flareon9/flareon9-t8-challenge-description.webp)
</figure>

First off, _ouch_. Second, this challenge started off strong with a binary programed to sleep for 43.200 seconds before it would continue:

<figure markdown="span">
	![Flare-on 9 T8 sleep syscall](./images/flareon9/flareon9-t8-sleep-syscall.webp)
</figure>

<figure markdown="span">
	![Flare-on 9 T8 sleep syscall assembly view](./images/flareon9/flareon9-t8-sleep-syscall-assembly.webp)
</figure>

As can be seen by my comment, the sleep syscall here didn't do much for the program itself, and can thus be patched out by changing the OP code to a `JMP` from a `JZ` instead:

<figure markdown="span">
	![Flare-on 9 T8 patched sleep syscall](./images/flareon9/flareon9-t8-patched-sleep-syscall-assembly.webp)
</figure>

However, with this patched sleep I wasn't able to get much further. Looking at the code following this showed a series of `uStack` variable containing some hex with `0x00` between them. These turned out to be a `wchar_t[16]` array as Windows uses UTF-16, rather than 8 making any UTF-8 string a series of characters with `0x00` to pad the length to the next character.

Doing this type change changed the data from the following to a neat `uStack72` array which I later renamed to `possible_domain` as it contained `flareon.com` in the name:

Original `uStack` array:

<figure markdown="span">
	![Flare-on 9 T8 ustack array before any changes](./images/flareon9/flareon9-t8-ustack-raw.webp)
</figure>

`uStack` array changed to `wchar_t[16]` and renamed to `possible_domain`:

<figure markdown="span">
	![Flare-on 9 T8 ustack array after changing types to wchar_t[16] in Ghidra decompile](./images/flareon9/flareon9-t8-ustack-renamed-new-type.webp)
</figure>

How I managed to get `flareon.com` from this was a hint below this array that did an XOR operation over these values.

The `straight_up_eight` value is decremented by 2 and uses it as its array index to do an XOR 17 operation over it. Doing this operation in Cyberchef with the string `0x7c007e0072003f` showed a string of `.com`:

<figure markdown="span">
	![Flare-on 9 T8 XOR 17 recipe in Cyberchef](./images/flareon9/flareon9-t8-xor17-domain.webp)
</figure>

This is where I/we got stuck for a while, messing around with the binary and the PCAPs it would create. It seemed that the PCAP showed random numbers for each POST request. We tracked this random number generator back to the function `FUN_00421ff4`:

<figure markdown="span">
	![Flare-on 9 T8 UA number generator](./images/flareon9/flareon9-t8-ua-number-generator.webp)
</figure>

Looking up those two hex values as their decimal counter parts (214013 & 2531011 respectively) returned a hit by Wikipedia for a linear congruential generator with the following table of parameters in use:

<figure markdown="span">
	![Wikipedia screenshot regarding commonly used values in linear congruential generators](./images/flareon9/flareon9-T8-LCG-MS.webp)
  <figcaption>Source: https://en.wikipedia.org/wiki/Linear_congruential_generator#Parameters_in_common_use</figcaption>
</figure>

Patching this function to always return the value from the PCAP (11950) makes this function look the following:

<figure markdown="span">
	![Flare-on 9 T8 patched LGC value](./images/flareon9/flareon9-t8-patched-lcg-value.webp)
</figure>

Yassir wrote the following Python server to emulate the server and how it responded in the PCAP:

```python title="Web server to emulate expected responses" linenums="1"
from http.server import BaseHTTPRequestHandler, HTTPServer
import time

class RequestHandler(BaseHTTPRequestHandler):
    sys_version = ""
    server_version = "Apache On 9 "
    def date_time_string(self,timestamp=0):
        return "Tue, 14 Jun 2022 16:14:36 GMT"
    def do_POST(self):
        if "; CLR" in str(self.headers):
            print("Request 2")
            message = "F1KFlZbNGuKQxrTD/ORwudM8S8kKiL5F906YlR8TKd8XrKPeDYZ0HouiBamyQf9/Ns7u3C2UEMLoCA0B8EuZp1FpwnedVjPSdZFjkieYqWzKA7up+LYe9B4dmAUM2lYkmBSqPJYT6nEg27n3X656MMOxNIHt0HsOD0d+"
        else:
            print("Request 1")
            message = "TdQdBRa1nxGU06dbB27E7SQ7TJ2+cd7zstLXRQcLbmh2nTvDm1p5IfT/Cu0JxShk6tHQBRWwPlo9zA1dISfslkLgGDs41WK12ibWIflqLE4Yq3OYIEnLNjwVHrjL2U4Lu3ms+HQc4nfMWXPgcOHb4fhokk93/AJd5GTuC5z+4YsmgRh1Z90yinLBKB+fmGUyagT6gon/KHmJdvAOQ8nAnl8K/0XG+8zYQbZRwgY6tHvvpfyn9OXCyuct5/cOi8KWgALvVHQWafrp8qB/JtT+t5zmnezQlp3zPL4sj2CJfcUTK5copbZCyHexVD4jJN+LezJEtrDXP1DJNg=="
        self.protocol_version = "HTTP/1.0"
        self.send_response(200)
        self.end_headers()
        self.wfile.write(bytes(message, "utf8"))
        return
def run():
    server = ('', 80)
    httpd = HTTPServer(server, RequestHandler)
    httpd.serve_forever()
run()
```

Running the binary now shows a pop-up saying "You're a machine!!!" in the form of a Fatal Application Exit:

<figure markdown="span">
	![Flare-on 9 T8 application error](./images/flareon9/flareon9-t8-application-error.webp)
</figure>

And this is where we then got stuck again for a _long_ time, trying to figure out what the binary did with the base 64 string sent by the server.

Turns out, we hade solved the challenge at this point an we hadn't figured that out just yet. The flag got stored somewhere in the application memory when it ran and triggered the exception, all you had to do was look for it:

<figure markdown="span">
	![Flare-on 9 T8 flage in process memory](./images/flareon9/flareon9-t8-flag-in-memory.webp)
</figure>

## Challenge 6: à la mode

<figure markdown="span">
	![Flare-on 9 alamode challenge description](./images/flareon9/flareon9-alamode-challenge-description.webp)
</figure>

At first glance this was a pure .NET challenge for which you had to perhaps develop some of your own code as the section that did something with the flag was missing:

<figure markdown="span">
	![Flare-on 9 alamode DnSpy overview](./images/flareon9/flareon9-alamode-dnspy.webp)
</figure>

And the empty flag section:

<figure markdown="span">
	![Flare-on 9 alamode empty function section](./images/flareon9/flareon9-alamode-empty-section.webp)
</figure>

The other section of this challenge, and how I solved it, was dissecting the binary inside Ghidra and figuring out what it did during its runtime.

First, the function named `dllmain_dispatch` at `0x100016e4` makes a call to the function `FUN_10001163`:

<figure markdown="span">
	![Flare-on 9 alamode Ghidra view of dllmain_dispatch](./images/flareon9/flareon9-alamode-ghidra-main.webp)
</figure>

`FUN10001163` makes a call to renamed function `FUN_100012f1` -> `FUN_all_available_functions_list` if there are two parameters:

<figure markdown="span">
	![Flare-on 9 alamode call to function index](./images/flareon9/flareon9-alamode-ghidra-call-to-index.webp)
</figure>

This `FUN_100012f1` contains a list of definitions and calls to similar functions that, at first, appear obfuscated or transformed into something illegible:

<figure markdown="span">
	![Flare-on 9 alamode function index overview](./images/flareon9/flareon9-alamode-ghidra-function-index.webp)
</figure>

When checking any of the `FUN_100014ae` functions it becomes clear that the input is processed with XOR 17:

<figure markdown="span">
	![Flare-on 9 alamode Ghidra XOR 17 function](./images/flareon9/flareon9-alamode-ghidra-xor17-function.webp)
</figure>

We can now decode all these weird char strings with XOR 17 in Cyberchef to get the original name for these functions:

<figure markdown="span">
	![Flare-on 9 alamode Cyberchef XOR 17 recipe](./images/flareon9/flareon9-alamode-cyberchef-xor17.webp)
</figure>

Doing this for all the names changes `FUN_100014ae` to look the following:

<figure markdown="span">
	![Flare-on 9 alamode Ghidra function names](./images/flareon9/flareon9-alamode-ghidra-decoded-names.webp)
</figure>

The following hours I spent renaming variables in various functions according to Microsoft documentation to match their originals.

In the end, the function and sub-functions of value were stored at `FUN_1000100` renamed as `FUN_processes_buffer` containing the following code:

<figure markdown="span">
	![Flare-on 9 alamode Ghidra buffer processor code](./images/flareon9/flareon9-alamode-ghidra-buffer-processor.webp)
</figure>

The first named function `FUN_init_substitution_table` or `FUN_100011ef` contained the following code that sets the substitution table for the crypto algorithm:

<figure markdown="span">
	![Flare-on 9 alamode Ghidra init substitution table code](./images/flareon9/flareon9-alamode-ghidra-init-sub-table.webp)
</figure>

And a `FUN_decrypt` function that took the contents of the buffer and decrypted them, using the state reached by the substitution table:

<figure markdown="span">
	![Flare-on 9 alamode Ghidra decrypt function](./images/flareon9/flareon9-alamode-ghidra-decrypt.webp)
</figure>

There were now two ways of solving this, either patching enough of the binary to run it and somehow get the password out of it. Or, option two: rewriting the functions this binary did to decode the buffer and the return us the contents. I opted for number two since it seemed easier to me at the time. Little did I know, that attempting to reconstruct working C++ code from a decompiled source sucks and is much more difficult than I first thought.

In the end, with some help from mryoranimo explaining compiler errors, I used the following C++ code to solve this challenge, emulating its actions and not having to check for a password at all:

```cpp linenums="1" title="C++ code to solve challenge without password checks"
#include <cstdio>
#include <cstdint>

/*
    Credits to mryoranimo for this C++ code <3
*/

int main() {
    uint8_t ARRAY_8bytes[8] = { 0x55, 0x8b, 0xec, 0x83, 0xec, 0x20, 0xeb, 0xfe, };
	uint8_t dest[12] = { 0x3e, 0x39, 0x51, 0xfb, 0xa2, 0x11, 0xf7, 0xb9, 0x2c, 0x00, 0x00, 0x00, };
	uint8_t flag[32] = { 0xe1, 0x60, 0xa1, 0x18, 0x93, 0x2e, 0x96, 0xad, 0x73, 0xbb, 0x4a, 0x92, 0xde, 0x18, 0x0a, 0xaa, 0x41, 0x74, 0xad, 0xc0, 0x1d, 0x9f, 0x3f, 0x19, 0xff, 0x2b, 0x02, 0xdb, 0xd1, 0xcd, 0x1a, 0x00, };

    uint8_t iVar1;
	uint8_t iVar2;
    uint32_t offset;
    uint8_t iVar3;
	uint8_t uVar3;
	uint32_t iVar4;
	uint32_t uVar4;
	uint32_t uVar2;

    /*
        named local_40c in Ghidra
        size is 258 because FUN_init_substitution_table does + 2 while i < 256
        i will reach 256 and run again, making it
    */
    uint8_t table[258] = { 0 };

    printf("FLAREON9 - Challenge 6 a la mode\n\n");

    table[0] = 0;
    table[1] = 0;

    offset = 0;

    do {
        table[offset + 2] = offset;
        table[offset + 2 + 1] = offset + 1;
        table[offset + 2 + 2] = offset + 2;
        table[offset + 2 + 3] = offset + 3;
        offset = offset + 4;
    } while (offset < 256);

	iVar2 = 0;
    uVar3 = 0;
    offset = 0;
	iVar4 = 0;

	do {
		offset = table[iVar4 + 2];
		uVar3 = (uint8_t) (ARRAY_8bytes[iVar2] + (char) uVar3 + (char) offset);
		table[iVar4 + 2] = table[uVar3 + 2];
		iVar1 = iVar2 + 1;
		table[uVar3 + 2] = offset;
		iVar4 = iVar4 + 1;
		iVar2 = 0;
		if (iVar1 < 8) {
			iVar2 = iVar1;
		}
	} while (iVar4 < 256);

	uint32_t i;
	uint8_t uVar1;
	char cVar3;

	i = 0;
	uint8_t local_4;
	local_4 = table[1];
	uVar4 = table[0];

	if (0 < 9) {
		do {
			uVar4 = (char) uVar4 + 1;
			uVar1 = table[uVar4 + 2];
			cVar3 = (char)uVar1;
			local_4 = (uint8_t) ((char) uVar1 + (char)local_4);
			uVar2 = table[local_4 + 2];
			table[uVar4 + 2] = uVar2;
			table[local_4 + 2] = uVar1;
			dest[i] = dest[i] ^ table[(uint8_t)(cVar3 + (char) uVar2) + 2];
			i = i + 1;
		} while (i < 9);
	}

	table[0] = uVar4;
	table[1] = local_4;

	printf("encryption password: %s\n\n", dest);

	i = 0;
	local_4 = table[1];
	uVar4 = table[0];

	if (0 < 0x1f) {
		do {
			uVar4 = uVar4 + 1;
			uVar1 = table[uVar4 + 2];
			cVar3 = (char)uVar1;
			local_4 = (uint8_t) ((char) uVar1 + (char)local_4);
			uVar2 = table[local_4 + 2];
			table[uVar4 + 2] = uVar2;
			table[local_4 + 2] = uVar1;
			flag[i] = flag[i] ^ table[(uint8_t)(cVar3 + (char) uVar2) + 2];
			i = i + 1;
		} while (i < 0x1f);
	}

	table[0] = uVar4;
	table[1] = local_4;

    printf("flag:\n");
	for (i = 0; i < 0x1f; i++) {
		printf("%c", flag[i]);
	}
    printf("\n");

    return 0;
}
```

Then compiling this with `cl /EHsc main.cpp` and running it showed both the password, and the flag:

<figure markdown="span">
	![Flare-on 9 alamode flag output](./images/flareon9/flareon9-alamode-challenge-flag.webp)
</figure>

I would later find out that the challenge name is a reference to [mixed mode assembly][ms-mm-assembly], a programming concept I hadn't come across until then.

## Challenge 7: anode

<figure markdown="span">
	![Flare-on 9 anode challenge description](./images/flareon9/flareon9-anode-challenge-description.webp)
</figure>

This challenge did my head in for a while. Cryptography is a field that fascinates me, but I don't grasp a lot of the math in it, let alone enough to break or reverse that done by it to get back to the original state/input.

At first there was a _big_ binary that took a while for Ghidra to process; its size was 54 **M**B. Capital M for Mega.

<figure markdown="span">
	![Flare-on 9 anode big boy binary](./images/flareon9/flareon9-anode-challenge-binary-size.webp)
</figure>

When running the binary it asked for a flag, and would return `try again` when the flag wasn't valid, I assumed at the time.

<figure markdown="span">
	![Flare-on 9 anode binary output](./images/flareon9/flareon9-anode-binary-output.webp)
</figure>

Running `FLOSS` over the binary returned an error that there were too many lines for `FLOSS` to process; a good indicator of what was to come later down the line. Within the output of `FLOSS` a section of JavaScript handling the flag you entered into it came to my attention:

```js title="FLag checking code" linenums="1"
readline.question(`Enter flag: `, flag => {
  readline.close();
  if (flag.length !== 44) {
    console.log("Try again.");
    process.exit(0);
  var b = [];
  for (var i = 0; i < flag.length; i++) {
    b.push(flag.charCodeAt(i));
  // something strange is happening...
  if (1n) {
    console.log("uh-oh, math is too correct...");
    process.exit(0);
  var state = 1337;
  while (true) {
    state ^= Math.floor(Math.random() * (2**30));
    switch (state) {
      case 306211:
        if (Math.random() < 0.5) {
          b[30] -= b[34] + b[23] + b[5] + b[37] + b[33] + b[12] + Math.floor(Math.random() * 256);
          b[30] &= 0xFF;
        } else {
          b[26] -= b[24] + b[41] + b[13] + b[43] + b[6] + b[30] + 225;
          b[26] &= 0xFF;
        }
        state = 868071080;
        continue;
      case 311489:
        if (Math.random() < 0.5) {
          b[10] -= b[32] + b[1] + b[20] + b[30] + b[23] + b[9] + 115;
          b[10] &= 0xFF;
        } else {
          b[7] ^= (b[18] + b[14] + b[11] + b[25] + b[31] + b[21] + 19) & 0xFF;
        }
        state = 22167546;
        continue;
      case 755154:
```

With the above JavaScript we can now deduct that the flag needs to be 44 characters in length to pass the initial check, but we also need to patch it to prevent the binary from exiting and skipping its actions.

mryoranimo and his [galaxy-brain][gbrain] spent that evening reading the Nexe compiler documentation and creating a Python script via which we would could replace the user code section of the `anode.exe` binary with whatever JavaScript code we wanted to.

Usage:

```
Usage: ./patch.py anode.exe resource.js anode-new.exe
```

Python script:

```python title="Replace user-code in Nexe compiled binary" linenums="1"
#!/usr/bin/env python3

import sys
import os
import struct
from math import floor

FOOTER_STRUCT = "<dd"
FOOTER_STRUCT_SZ = struct.calcsize(FOOTER_STRUCT)

FOOTER_MAGIC = b"<nexe~~sentinel>"

def main(args):
    infile = args[0]
    patchfile = args[1]
    outfile = args[2]

    infile_sz = os.path.getsize(infile)

    with open(infile, "rb") as infile_stream:
        infile_stream.seek(infile_sz - FOOTER_STRUCT_SZ)
        content_sz, resource_sz = [ floor(x) for x in struct.unpack_from(FOOTER_STRUCT, infile_stream.read()) ]

        # seek back to the packed footer
        infile_stream.seek(0 - len(FOOTER_MAGIC) - FOOTER_STRUCT_SZ, os.SEEK_CUR)

        # from there, seek back to the beginning of the content payload
        infile_stream.seek(0 - (content_sz + resource_sz), os.SEEK_CUR)

        # now we can read both the content and resource payloads
        content_buf = infile_stream.read(content_sz)
        resource_buf = infile_stream.read(resource_sz)

    print("Content size: {}".format(content_sz))
    print("Resource size: {}".format(resource_sz))

    print("Retrieved content size: {}".format(len(content_buf)))
    print("Retrieved resource size: {}".format(len(resource_buf)))

    with open(patchfile, "rb") as patchfile_stream:
        patch = patchfile_stream.read()

    with open(outfile, "wb") as outfile_stream:
        with open(infile, "rb") as infile_stream:
            remainder = infile_sz - content_sz - resource_sz - len(FOOTER_MAGIC) - FOOTER_STRUCT_SZ
            while remainder > 0:
                buf = infile_stream.read(min(remainder, 8192 * 1024))
                outfile_stream.write(buf)
                remainder -= len(buf)
                print(remainder)

        patch_sz = len(patch)
        content_buf = content_buf.replace(str(resource_sz).encode(), str(patch_sz).encode())
        outfile_stream.write(content_buf)
        outfile_stream.write(patch)
        outfile_stream.write(FOOTER_MAGIC)
        outfile_stream.write(struct.pack(FOOTER_STRUCT, len(content_buf), patch_sz))

if __name__ == "__main__":
    main(sys.argv[1:])
```

Another thing that we concluded from the JavaScript is that the PRNG used in this binary is probably seeded, and will thus always produce the same results with the same inputs. we concluded this from the JavaScript because it contains statements for the math either being "too correct" or "math.random() is too random".

mryoranimo explained that it would now be possible to revert the mutations done by the binary if we'd get all the mutations that it did in order, and then walked those back. To get all the states the binary went through we changed the JavaScript case code from executing anything to printing it to STDOUT instead.

Going from this:

```js linenums="1"
switch (state) {
    case 306211:
        if (Math.random() < 0.5) {
            b[30] -= b[34] + b[23] + b[5] + b[37] + b[33] + b[12] + Math.floor(Math.random() * 256);
            b[30] &= 0xFF;
        } else {
            b[26] -= b[24] + b[41] + b[13] + b[43] + b[6] + b[30] + 225;
            b[26] &= 0xFF;
        }
        state = 868071080;
        continue;
```

To now this:

```js linenums="1"
switch (state) {
      case 306211:
        if (Math.random() < 0.5) {
          statestring += "b[30] -= b[34] + b[23] + b[5] + b[37] + b[33] + b[12] " + Math.floor(Math.random() * 256);
          statestring += "b[30] &= 0xFF;"
        } else {
          statestring += "b[26] -= b[24] + b[41] + b[13] + b[43] + b[6] + b[30] + 225";
          statestring += "b[26] &= 0xFF;"
        }
        state = 868071080;
        continue;
```

To do this for all 1024 states I used some Regex magic `^(\s+)(b[\d+].*;)$` and substituting it with `$1state_transition_string += "$2";`. For the remaining substitutions a find-and-replace for `Math.floor(Math.random() * 256` with `" + Math.floor(Math.random() * 256) + "` would suffice. Now running the binary in a terminal returned the following:

```powershell
LARE 10/15/2022 7:38:41 PM
PS C:\Users\IEUser\Documents\FLAREON9\07_anode\Code > ..\binary\anode-show-states.exe
Enter flag: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
b[29] -= b[37] + b[23] + b[22] + b[24] + b[26] + b[10] + 7;b[29] &= 0xFF;b[39] += b[34] + b[2] + b[1] + b[43] + b[20] + b[9] + 79;b[39] &= 0xFF;b[19] ^= (b[26] + b[0] + b[40] + b[37] + b[23] + b[32] + 255) & 0xFF;b[28] ^= (b[1] + b[23] + b[37] + b[31] + b[43] + b[42] + 245) & 0xFF;b[39] += b[42] + b[10] + b[3] + b[41] + b[14] + b[26] + 177;b[39] &= 0xFF;b[9] -= b[20] + b[19] + b[22] + b[5] + b[32] + b[35] + 151;b[9] &= 0xFF;b[14] -= b[4] + b[5] + b[31] + b[15] + b[36] + b[40] + 67;b[14] &= 0xFF;b[33] += b[25] + b[12] + b[14] + b[34] + b[4] + b[36] + 185;b[33] &= 0xFF;
[snip]
b[22] += b[16] + b[18] + b[7] + b[23] + b[1] + b[27] + 50;b[22] &= 0xFF;b[39] += b[18] + b[16] + b[8] + b[19] + b[5] + b[23] + 36;b[39] &= 0xFF;
Try again.
```

Leaving us with about 1700 odd lines of JavaScript we need to reverse the order of, that looked a little like this:

```js linenums="1"
b[29] -= b[37] + b[23] + b[22] + b[24] + b[26] + b[10] + 7;
b[29] &= 0xff;
b[39] += b[34] + b[2] + b[1] + b[43] + b[20] + b[9] + 79;
b[39] &= 0xff;
b[19] ^= (b[26] + b[0] + b[40] + b[37] + b[23] + b[32] + 255) & 0xff;
b[28] ^= (b[1] + b[23] + b[37] + b[31] + b[43] + b[42] + 245) & 0xff;
b[39] += b[42] + b[10] + b[3] + b[41] + b[14] + b[26] + 177;
b[39] &= 0xff;
b[9] -= b[20] + b[19] + b[22] + b[5] + b[32] + b[35] + 151;
b[9] &= 0xff;
b[14] -= b[4] + b[5] + b[31] + b[15] + b[36] + b[40] + 67;
b[14] &= 0xff;
b[33] += b[25] + b[12] + b[14] + b[34] + b[4] + b[36] + 185;
b[33] &= 0xff;
```

Reversing the document itself was easy with the `tac` command. Reversing the math done in the document took a _little while longer_ but we got there in the end.

Say we have the following mutation that happens at the end of the chain:

```js
b[39] &= 0xff;
b[39] += b[18] + b[16] + b[8] + b[19] + b[5] + b[23] + 36;
```

The math done here can be reversed/undone by doing the math in reverse order:

```js
b[39] = ((211 + 106 + 66 + 68 + 102 + 38 + 36) & 255)
b[39] += 627 & 255
212 = (x + 627) & 255
x = 112 ( 627 & 255 = 100) ((100 +112) & 255 = 212)
```

To then do this for all the 1024 or so cases can be done with the following two regex patterns:

```
  # flip - with +
  search: ^(b\[\d+\]) -= (.*);$
  replace: $1 = ($1 + ($2)) & 255;

  # flip + with -
  search: ^(b\[\d+\]) += (.*);$
  replace: $1 = ($1 - ($2)) & 255;
```

Which creates something a little like this:

```js
b[39] = (b[39] - (b[18] + b[16] + b[8] + b[19] + b[5] + b[23] + 36)) & 255;
b[22] = (b[22] - (b[16] + b[18] + b[7] + b[23] + b[1] + b[27] + 50)) & 255;
b[34] = (b[34] - (b[35] + b[40] + b[13] + b[41] + b[23] + b[25] + 14)) & 255;
b[21] = (b[21] - (b[39] + b[6] + b[0] + b[33] + b[8] + b[40] + 179)) & 255;
b[11] = (b[11] + (b[32] + b[8] + b[9] + b[34] + b[39] + b[19] + 185)) & 255;
b[19] ^= (b[0] + b[35] + b[14] + b[30] + b[21] + b[33] + 213) & 0xff;
b[40] = (b[40] - (b[13] + b[3] + b[43] + b[31] + b[22] + b[25] + 49)) & 255;
b[17] ^= (b[41] + b[14] + b[43] + b[6] + b[7] + b[28] + 196) & 0xff;
```

Putting this into a JavaScript program looked the following:

```js linenums="1"
var b = [
  106, 196, 106, 178, 174, 102, 31, 91, 66, 255, 86, 196, 74, 139, 219, 166,
  106, 4, 211, 68, 227, 72, 156, 38, 239, 153, 223, 225, 73, 171, 51, 4, 234,
  50, 207, 82, 18, 111, 180, 212, 81, 189, 73, 76,
];
b[39] = (b[39] - (b[18] + b[16] + b[8] + b[19] + b[5] + b[23] + 36)) & 255;
b[22] = (b[22] - (b[16] + b[18] + b[7] + b[23] + b[1] + b[27] + 50)) & 255;
b[34] = (b[34] - (b[35] + b[40] + b[13] + b[41] + b[23] + b[25] + 14)) & 255;
b[21] = (b[21] - (b[39] + b[6] + b[0] + b[33] + b[8] + b[40] + 179)) & 255;
b[11] = (b[11] + (b[32] + b[8] + b[9] + b[34] + b[39] + b[19] + 185)) & 255;
b[19] ^= (b[0] + b[35] + b[14] + b[30] + b[21] + b[33] + 213) & 0xff;
b[40] = (b[40] - (b[13] + b[3] + b[43] + b[31] + b[22] + b[25] + 49)) & 255;
b[17] ^= (b[41] + b[14] + b[43] + b[6] + b[7] + b[28] + 196) & 0xff;
b[38] = (b[38] - (b[20] + b[30] + b[31] + b[8] + b[37] + b[33] + 54)) & 255;
b[19] ^= (b[3] + b[30] + b[17] + b[15] + b[13] + b[18] + 241) & 0xff;
b[30] = (b[30] + (b[25] + b[34] + b[36] + b[6] + b[41] + b[11] + 108)) & 255;
// snip
b[39] = (b[39] - (b[34] + b[2] + b[1] + b[43] + b[20] + b[9] + 79)) & 255;
b[29] = (b[29] + (b[37] + b[23] + b[22] + b[24] + b[26] + b[10] + 7)) & 255;

console.log(String.fromCharCode(...b));
```

Running this returned the flag:

<figure markdown="span">
	![Flare-on 9 anode flag ](./images/flareon9/flareon9-anode-flag.webp)
</figure>

And to make sure, entering this flag into the challenge binary returned `congrats`:

<figure markdown="span">
	![Flare-on 9 anode flag entered into original challenge binary](./images/flareon9/flareon9-anode-flag-input-in-original-binary.webp)
</figure>

## Challenge 8: Backdoor

From my time spent on Twitter lurking at what others had to mention about the challenges, it seemed that Challenge 8 was the hardest one of the bunch. This turned out correct as neither Yassir, mryoranimo nor I managed to solve it.

I didn't actually look at this challenge a whole lot, as at the time when we got to it I lost a bit of motivation to work on _only_ Flare-on day-in-and-out.

<figure markdown="span">
	![Flare-on 9 backdoor challenge description](./images/flareon9/flareon9-backdoor-challenge-description.webp)
</figure>

## Conclusions

All in all. I made it _way_ further than I could have ever dreamed of going into flare-on pretty much blind and without any prior experience. Battling my way through the 7 challenges that I managed to complete felt rewarding once getting the hints in the challenge name or description and understanding what I had been looking at.With this positive experience behind me, I am looking forward to what next year's Flare-on has in store for me.

[kladblokje-twitter]: https://twitter.com/kladblokje_88
[yt-shylilly]:        https://www.youtube.com/watch?v=3TMX3K_7klY
[diffchecker]:        https://diffchecker.com
[gh-floss]:           https://github.com/mandiant/flare-floss
[x86-ret]:            https://c9x.me/x86/html/file_module_x86_id_280.html
[ms-mm-assembly]:     https://learn.microsoft.com/en-us/cpp/dotnet/mixed-native-and-managed-assemblies?redirectedfrom=MSDN&view=msvc-170
[gbrain]:             https://en.wiktionary.org/wiki/galaxy-brain#English
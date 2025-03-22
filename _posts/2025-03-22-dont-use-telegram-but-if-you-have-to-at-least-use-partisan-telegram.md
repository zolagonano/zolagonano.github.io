---
title: "Don't use Telegram, but if you have to, at least use Partisan Telegram"
layout: post
series: Privacy
---


Telegram is really close to the worst when it comes to privacy, security, and anonymity, which highly matters if you are an activist, a protester, a journalist, or even a normal paranoid human being.

Telegram is a platform that keeps on animating its emoji reactions in every update and milking users for money by giving them a "premium" subscription that essentially provides nothing "premium" in exchange. I have had Telegram since it came out on Android, and that’s about 10 years ago, but apparently, that wasn't enough time for Telegram to add E2EE to their platform.

Of course, it provides so-called "Secret Chats," but what is the use of it if it is only available on mobile clients and its encryption is self-rolled, provoking one of the most important laws in cryptography, which states explicitly: "DO NOT ROLL YOUR OWN CRYPTO."

But besides its lack of proper E2EE, you might have to use it. I have to use it. Just because everyone I know uses it. And that’s a huge reason, believe it or not—you cannot go live on your own island forever. I have tried, and I have failed.

With all this hate speech I did on Telegram, it is time to introduce a client called "Partisan Telegram" that not only fixes the issues with E2EE but also provides some other kind of security to it.

Partisan Telegram brings operational security to Telegram, and by using it, you are still trusting Telegram with your data, which is not something you should do, honestly. Do not trust anyone with your data, especially if they can profit off your data or be pressured to hand out your data (be realistic—no one would shut down their platform in a country for you, let alone go to jail for you).

Partisan Telegram adds some privacy and security features that can be life-saving for protesters and activists in some parts of the world where freedom is far from a luxury, in places where expressing it is considered a crime and punished severely.

I like to keep this post short, so I will just explain how it works and what it offers.

## How It Works

Partisan Telegram tries its best to obfuscate its existence so it wouldn't put you into more trouble if you're caught with it. So everything feels and looks like vanilla Telegram.

It uses the lock feature of Telegram, where you put a 4-digit passcode to access your Telegram. By using Partisan Telegram, you can set multiple passcodes, which can then be programmed to do certain things when entered. This is its main functionality.

For example, you can set the passcode "1234" to show only one account that you have nothing on. Or you can program it to send an alert message to your contacts when activated or to log off your accounts on all devices. And you will have a main passcode that will give you full access to your Telegram.

But how would this be useful? Imagine you're caught mid-protest and someone is investigating your phone. You can just give them the fake passcode that you have programmed to log out your accounts and remove any evidence from the device without them noticing while alerting your friends who might have been involved. The scenarios can be endless.

## What It Offers

Partisan Telegram's main focus is your OPSEC, as it cannot add E2EE to Telegram without breaking its backward compatibility with other versions of Telegram. And it is relatively good at it.

### Configurability

Partisan Telegram has great configurability. You can set endless fake passcodes and configure each to do a certain thing for a specific situation.

You can set a timer to go off or set a number of failed attempts to trigger a fake passcode, which will then trigger other events you have programmed it to do.

It can also take pictures when entering a wrong or fake passcode, which you can later check to see if someone has attempted to unlock your Telegram or not.

### Triggering Killswitch by Receiving a Message

One main feature it provides is that the fake passcode actions or killswitch can be triggered if you receive a specific message.

This can be very useful if you lost your phone, or it got stolen or taken from you. You can remotely delete all your accounts from that phone and send an SOS message to your key contacts.

### Providing Additional Security Measures

It also provides some other security measures, like brute-force protection for the passcodes, which Telegram itself doesn't. With brute-force protection, the time between login attempts will increase every time.

It can also be set to delete cache and drafts on exit, which the original Telegram doesn't offer.

It can prevent accidental clicks by double-checking with you when you accidentally touch the subscribe button or send a reaction to a message.

### Delete After Read

It provides automatic delete-after-read for specific messages. You can send a message with a timer, and it will be deleted after the timer has passed. It is not like Telegram’s auto-delete, which deletes all messages from both sides after a period of time and is also visible when enabled or disabled, which can be more suspicious in some cases.

Other than that, Partisan Telegram also provides auto-delete to delete all of your messages in a group chat automatically. This can come in handy if you need to delete your messages and leave. Although all group chat events, like deleting or editing messages, will be visible to the admins of the group chat for 72 hours.

### File Protection

Partisan Telegram can prevent information extraction from your phone by encrypting the data stored on the phone, including messages.

### Encrypted Group Chats

Partisan Telegram recently introduced an experimental feature for having an encrypted chat with a group of people, which can be useful for organizing.

## Other Perks

It can also bypass some of the annoying paywalls that Telegram has recently put up. For example, it will allow you to add more than three accounts without subscribing to Telegram Premium.

It can provide additional badges to certain accounts, helping you detect suspicious accounts and scammers before you interact with them.

## Security Precautions

You should research before using anything, especially when it doesn’t have audits and a well-known, reputed developer team. But Partisan Telegram is fully open-source and can be compared against the original Telegram codebase to see exactly what has been modified. You can compile your own version instead of trusting their binaries.

You can find Partisan Telegram at [https://github.com/wrwrabbit/Partisan-Telegram-Android](https://github.com/wrwrabbit/Partisan-Telegram-Android).

It is a good project for a good purpose, but if you can avoid Telegram and use something like Signal (which includes E2EE and is built with privacy in mind), do it.
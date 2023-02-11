---
title: "Speak Easy: From Speech to Text with AI"
date: 2022-12-25T10:18:59-06:00
draft: False
---

## Introduction

One of my ambitions is to share content more often, but I often struggle with proofreading and capturing my ideas. I believe that my content is not perfect enough to be shared with a wider audience, which hinders my ability to share it. When I do have an idea for a blog, I often don't have the time to record it because I'm in the middle of another activity. Even when I do have the time, I find it difficult to structure the content to a level that I feel is fit to share. To overcome these challenges and build a habit of sharing content, I plan to create a mobile app that captures speech and translates it to text. The app will then display the text on the screen and send it to OpenAI for proofreading and formatting. Once complete, the app can publish the content to GitHub.

## System Design

1. Mobile Client: The client is a mobile or web UI that will be responsible for recording the user and presenting to the user the text and audio representation of their recent recording.
1. Speech to text converter: The speech to text converter will be an API that will access the audio snippets stored by the client in a secure location and transcribe them into text.
1. Text editor: Reviews the transcribed text, edits, refines and structures it. This is not a UI component but an AI component that is focused on re-structuring and re-formatting the content.
1. Publisher: Synchronises and allows you to publish the content to github

## Setup

### Client

The client is a mobile app built using [Flutter](https://flutter.dev/), an open-source mobile application framework created by Google. Flutter enables developers to to build cross-platform mobile apps for Android and iOS using a single codebase. One of the key benefits of Flutter is its hot reload feature, which allows developers to quickly iterate on their code and see the changes in real-time. This makes it easier to fix bugs and add new features. Flutter also offers a wide range of built-in widgets and tools for building attractive and responsive user interfaces. Additionally, its support for testing and continuous integration will enable me to deliver a quality app when it is released.

This post will not be a deep dive into Flutter development. Time permitting, I will write another article that focuses on my journey into flutter development. For the client, we are looking at targetting Android and IoS.

The docs for flutter are pretty extensive and you can install flutter by following the directions [here](https://docs.flutter.dev/get-started/install). I am using macOS and the screen shots here-on will reflect the setup and instructions for a mac. `flutter doctor` is a command that validates your install and you should see something similar to the below screen

![Flutter ready for dev](https://i.imgur.com/TKHbHzg.png)
## Astarot - native XCM enhancing UI tool dedicated to Astar

![astarot](https://raw.githubusercontent.com/dudo50/astarot/main/src/assets/astarot.png)

## Contents

[1. Introduction](#1-introduction)<br />
[2. Overview](#2-overview)<br />
[3. Installation](#3-installation)<br />


# 1. Introduction
Application is deployed & able to be previewed on following link - [Try out Astarot on Netlify](https://639db68af6fa0409cc687d24--celebrated-melomakarona-c33425.netlify.app/#/)

There are not that many XCM UI enhancing tools dedicated to Astar. A⭐️ot is enhancing native polkadotXCM pallet and alows user to transfer native XCM in three different scenarios. This saves time to users and developers also. A⭐️ot do not require specific message route from you, it fills it for you from simple details you are required to provide (way less details than if you were to construct call manually). Astarot benefits from really simple and easy to understand UI. As a new user you can adapt very quickly.

# 2. Overview
The project allows user to start Application, that can right out of the box create native XCM transfers in three different scenarios. These scenarios are:
- From Parachain to Relay chain (Either from Shiden to Kusama or from Astar to Polkadot)
- From Relay chain to Parachain (Either from Kusama to Shiden or from Polkadot to Astar)
- From Parachain to Parachain (Either from Shiden or Astar to any Parachain connected via same Relay chain)

Every screen is fitted notifications for important actions (eg. some detail not filled, user not logged in or transaction hash once transaction is submitted, information about block finalization).

A⭐️ot's main advantages can be sumarized into following points:
- You can execute native XCM calls from simple user friendly interface. 
- It is super fast to fill & send calls you need. 
- Your data is never stored or sent anywhere. 
- This project is strictly open source.
- You can switch between **Shiden & Kusama** or **Astar & Polkadot** with a simple click on switch.
- In transfer scenario Parachain to Parachain Astarot automatically queries connected nodes for both of your options (Polkadot connected nodes or Kusama connected nodes)

Instead of filling lengthy and confusing polkadotXCM pallet calls Astarot fills and constructs them for you. All this without requiring as many details as you would have to fill in original call.

In the picture below on the left is the amount of data Astarot requires & on right is the amount of data you would have to fill to create call necessary for XCM transfer. This is just one example from all 3 screens. Feel free to try others yourself. You can click on the image to see it in full resolution.
![imgonline-com-ua-twotoone-5Ietca8rh9TW2Z5](https://user-images.githubusercontent.com/55763425/197884670-d5ac2742-3bf5-4dbc-b656-66c0a0f96662.jpg)


# 3. Installation
Installation for local development is super easy.

#### Clone github repository via
```
git clone https://github.com/dudo50/astarot.git
```

#### Install dApp dependencies & packages
```
pnpm i
```

#### Start dApp for development
```
pnpm run serve
```

#### Build dApp for release
```
pnpm run build
```
# Project achievements
- Astar XCM bounty Polkadot hackathon Latam [Bounty link](https://github.com/AstarNetwork/AstarBounties/pull/17)

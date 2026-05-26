---
title: Gameplay Events
permalink: /docs/gas/gameplay_events/

toc: true
toc_sticky: true
sidebar:
  nav: gas_docs
---
Generally, when you need to notify that something happens during gameplay and let any other system react to the event, you use delegates. They allow you to broadcast an event without worrying about who is actually listening to it.

Gameplay Events are a similar concept, but specifically designed to be deeply integrated with the Gameplay Ability System. You send gameplay events to notify gameplay abilities that something has happened, along with any relevant information that can be used to react accordingly.

Events are sent using a gameplay tag as a the event identifier, and a payload structure that contains any additional data required. The payload is optional, as many abilities can simply react to a plain named event.

## 8.1 Sending Gameplay Events
PENDING

## 8.2 Activating Abilities with Gameplay Events
PENDING

## 8.3 Reacting to Gameplay Events
PENDING
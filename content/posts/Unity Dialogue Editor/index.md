---
title: "Unity Dialogue Editor"
summary: "A visual custom Unity editor interface for creating and managing branching dialogues."
categories: ["Post","Blog",]
tags: ["post","unity"]
showSummary: true
date: 2023-11-01
draft: false
---

## Class Diagram

![Class Diagram](https://github.com/FutureWayne/cubism-mbti/assets/39150337/4e0c3c92-e149-402e-bf4d-93a4e2d818ee)

## Node-based Dialogue Editor
The dialogue editor in the project provides a visual interface for creating and managing branching dialogues. It's designed to be intuitive and user-friendly, allowing developers and writers to visualize and design complex conversations with multiple branching paths.
![Dialogue Editor](https://github.com/FutureWayne/cubism-mbti/assets/39150337/7a9e8e3a-3c5b-48c8-966f-f73543860299)


### DialogueEditor Class
- **Key Features:**
  - Node Representation: Each dialogue piece is represented as a node. Nodes can be dragged, linked, and edited directly in the editor window.
  - Node Linking: Nodes can be linked to represent the flow of the conversation. The editor provides visual bezier curves to represent these links.
  - Node Creation and Deletion: New nodes can be created and linked to existing nodes. Nodes can also be deleted, and any links to them are automatically cleaned up.
  - Event Handling: The editor handles various events like mouse clicks and drags.

### DialogueNode Class
- **Key Features:**
  - Speaker Information: Each node contains information about the speaker (player, NPC, or narrative).
  - Child Nodes: Each node can have multiple child nodes representing different conversation paths.
  - Status Requirements: Nodes have status requirements for accessibility based on game conditions.
  - Buff Effects: Nodes can have associated buff effects.

### Dialogue Class
- **Key Features:**
  - Node Management: Methods to get all nodes, the root node, and child nodes.
  - Node Creation and Deletion: In the editor, new nodes can be created, and existing nodes can be deleted.

## Player Conversant System
The PlayerConversant class plays a pivotal role in the dialogue system by managing the active dialogue for the player, handling the flow of conversation, and integrating with the UI system.

### Dialogue Management
- Maintains references to the current dialogue and node.
- Methods to start a new dialogue, move to the next node, and quit the dialogue.

### UI Integration
- Methods to retrieve the current node's text, speaker name, speaker type, background, audio clip, and personality type.
- The `OnConversationUpdated` event signals the UI system to update displayed content.

### Choice Handling
- Indicates if the player is presented with dialogue choices.
- Retrieves available choices and allows the player to select a choice.

### Buff Integration
- `TriggerEnterAction` and `TriggerExitAction` methods handle buffs associated with dialogue nodes.

## Data Oriented Buff System
- Designed to apply temporary effects or "buffs" based on dialogue choices.

### BuffData
- Represents a single buff with properties like ID, name, trait, duration, and modifiers.
- Implements the IBuffEffect interface to apply or remove buffs.

### BuffFactory
- Static class for creating and managing buff instances.
- Loads buffs and provides methods to retrieve them by ID.

### IBuffEffect
- Interface defining methods for applying and removing buffs.

### BuffCfgReader
- Utility class for importing buffs from a CSV file.

![BuffSystem](https://github.com/FutureWayne/cubism-mbti/assets/39150337/0a478a4c-c416-4eac-b1ab-b8a971b01be1)
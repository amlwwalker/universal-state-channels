# Universal State Channels

Universal State Channels (or USC) is a platform for building state channels using centralized or decentralized judges.

*What is a state channel?* A state channel is a way for two or more parties to maintain a state (any sequence of bytes) between them, being able to update it without the involvement of any third parties.

*What are state channels used for?* The most well-known use of state channels is in payment channels, a technique which allows two parties to pay each other by adjusting the balance of an escrow account or a special blockchain transaction. Payment channels can also be linked together, allowing payments to be securely transmitted across multiple untrusted parties. Participants in the channel exchanged signed state update transactions, and can close the channel at any time with confidence that the last valid update transaction will be honored. Because only the last update transaction is sent to the judge, they can be used to create very scalable systems. [Here's](http://www.jeffcoleman.ca/state-channels/) an easy explanation of state channels.

Payment channels, and any other state chanel, can be implemented easily on top of USC with a minimum of effort. Here's a 200 line Javascript example app implementing a simple payment channel.

USC Peer handles communication between the participants in the channel, and cryptographically checks all transactions for validity. The developer writing the channel's business logic interacts with a friendly HTTP API over localhost. No specialized knowledge of cryptography is required.

USC Judge is run by a trusted third party and checks the validity of a state channel's final update transaction. It also has an easy API, which the developer can use to write the business logic which reacts to the state contained in the final update transaction. For example, in a payment channel, the USC Judge could be run by a bank and permanently transfer money when the channel closed.

USC will also have blockchain adapters, which run alongside the USC Peer. Blockchain adapters relay transactions to a judge contract on a blockchain. This allows the trusted third party to be replaced by trust in a programmable blockchain, such as Ethereum, Tendermint, or Hyperledger.

## HTTP Peer API

### Accounts

Accounts correspond to identities known by a third party judge or a blockchain. Accounts embed the information for their judge.

#### List All Accounts

`accounts` returns a list of all accounts stored in the USC Peer.

GET `https://localhost:4456/accounts`

*Example response:*

```
[
  {
    "name": "AC7739 at SFFCU",
    "pubkey": "R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=",
    "privkey": "k4NkO3BNxNN8qsdPvsKv9AEJMP_IqIqluy77HLcN1gVHmVVWzzYzzSLk6lHfr1K0mpodLrUt34_NMJ9L7TPIAA==",
    "judge": {
      "name": "San Francisco Federal Credit Union",
      "pubkey": "xcYNnNW1oA9pB0LeQg_UCKw3FC8itnVq1csGrHdCV6o=",
      "address": "https://sanfranciscofcu.com/channels/"
    }
  },
  ...
]
```

#### Accounts by Pubkey

`accounts_by_pubkey` returns account with the specified pubkey.

*Example:*

GET `https://localhost:4456/accounts_by_id/R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=`

Response: An account, see above.



### Counterparties

#### List all counterparties

`counterparties` returns a list of all accounts of counterparties known to the USC Peer. Counterparties are peers that USC can start channels with. Counterparties embed the information for their judge.

*Example:*

GET `https://localhost:4456/counterparties`

Response:

```
[
  {
    "name": "AC2346 at SFFCU",
    "pubkey": "R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=",
    "judge": {
      "name": "San Francisco Federal Credit Union",
      "pubkey": "xcYNnNW1oA9pB0LeQg_UCKw3FC8itnVq1csGrHdCV6o=",
      "address": "https://sanfranciscofcu.com/channels/"
    }
  },
  ...
]
```

#### Counterparties by pubkey

`counterparties/pubkey/<pubkey>` returns the counterparty with the specified pubkey.

*Example:*

GET `https://localhost:4456/counterparties/pubkey/R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=`

Response: A counterparty, see above.



### Channels

Channels embed information about their account, their counterparty, and their judge.

#### List all channels

`channels` returns a list of all channels which accounts on the USC Peer participate in.

*Example:*

Request: GET `https://localhost:4456/channels`

Response:
```
[
  {
    "channelId": "8789678",
    "phase": "OPEN",
    "openingTx": {
      "channelId": "8789678",
      "pubkeys": [
        "R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=",
        "prNVb9C260wELZ3RYmrJ9TsZ_2NCGYcUBVZSSGHUsYQ="
      ],
      "state": "{\"R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=\":100,\"prNVb9C260wELZ3RYmrJ9TsZ_2NCGYcUBVZSSGHUsYQ=\":100}"
    },
    "lastFullUpdateTx": {
      "channelId": "8789678",
      "sequenceNumber": 7,
      "fast": false,
      "state": "{\"R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=\":105,\"prNVb9C260wELZ3RYmrJ9TsZ_2NCGYcUBVZSSGHUsYQ=\":95}"
    },
    "myProposedUpdateTx": {
      "channelId": "8789678",
      "sequenceNumber": 7,
      "fast": false,
      "state": "{\"R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=\":105,\"prNVb9C260wELZ3RYmrJ9TsZ_2NCGYcUBVZSSGHUsYQ=\":95}"
    },
    "theirProposedUpdateTx": {
      "channelId": "8789678",
      "sequenceNumber": 6,
      "fast": false,
      "state": "{\"R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=\":102,\"prNVb9C260wELZ3RYmrJ9TsZ_2NCGYcUBVZSSGHUsYQ=\":98}"
    },
    "followOnTxs": [],
    "judge": {
      "name": "San Francisco Federal Credit Union",
      "pubkey": "xcYNnNW1oA9pB0LeQg_UCKw3FC8itnVq1csGrHdCV6o=",
      "address": "https://sanfranciscofcu.com/channels/"
    },
    "account": {
      "name": "AC7739 at SFFCU",
      "pubkey": "R5lVVs82M80i5OpR369StJqaHS61Ld-PzTCfS-0zyAA=",
      "privkey": "k4NkO3BNxNN8qsdPvsKv9AEJMP_IqIqluy77HLcN1gVHmVVWzzYzzSLk6lHfr1K0mpodLrUt34_NMJ9L7TPIAA==",
      "judge": {
        "name": "San Francisco Federal Credit Union",
        "pubkey": "xcYNnNW1oA9pB0LeQg_UCKw3FC8itnVq1csGrHdCV6o=",
        "address": "https://sanfranciscofcu.com/channels/"
      }
    },
    "counterparty": {
      "name": "AC2346 at SFFCU",
      "pubkey": "prNVb9C260wELZ3RYmrJ9TsZ_2NCGYcUBVZSSGHUsYQ=",
      "judge": {
        "name": "San Francisco Federal Credit Union",
        "pubkey": "xcYNnNW1oA9pB0LeQg_UCKw3FC8itnVq1csGrHdCV6o=",
        "address": "https://sanfranciscofcu.com/channels/"
      }
    }
  },
  ...
]
```

### Channels by Id

`channels/<channelId>` returns the channel with the specified channelId.

*Example:*

Request: GET `https://localhost:4456/channels_by_id/8789678`

Response: A channel, see above

// "privkey": "9Am0PA0NPNeeHuyAb2ssNkuX0Q0UEzoqopPPAL28BIjFxg2c1bWgD2kHQt5CD9QIrDcULyK2dWrVywasd0JXqg=="
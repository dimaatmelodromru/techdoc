# sphinx-tribes

## Public and Private groups on Sphinx

Sphinx.chat clients and relay software support group communications in the form of *Private* (up to 10 members) and *Public* (unlimited) groups (also called *Tribes* on Sphinx). 
Since every message on Sphinx is end-to end encrypted, for *Private* groups, Admin's Relay takes care of encrypting every message sent by it's members to the group by each recepient's key end sending that message to the recepient directly (over the Lightning network).
For bigger, *Public* groups, the Group Admin's Relay publishes messages on an intermediary message broker server, **sphinx-tribes**, that other group members Relays connect to to read messages in the group. This allows for a virtually unlimited number of members in a *Public* group.
To keep the messaging network as decentralized as necessary, anyone can run a **sphinx-tribes** server.

![Tribes](https://github.com/stakwork/sphinx-tribes/raw/master/img/tribes.jpg)

### Discovering a tribe

To join a *Public* A user needs to know the following group parameters:

- Group UUID;
- URL of the Tribes server it is hosted on;
- Group Admin's public key.

**sphinx-tribes** provides users with a Web interface that publishes the necessary information about the groups it hosts.

![Tribes web UI screenshot](tribes.sphinx.chat.png)
*Example of Tribes web UI*

**tribes-server** displays hosted *Public* groups' names, avatars, descriptions, tags; every item expands on click to display additional group info:

- Admin name;
- Members count;
- Creation date;
- Price to join;
- Price per message;
- Message Stake *(not available as of this writing yet)*;
- Join link as
    + clickable link
    + text string
    + COPY control
    + QR code

**Join link example**
```
sphinx.chat://?action=tribe&uuid=XtDJASBiyLKY8f1fe7AOVts04oKbtJRiFoJgDH8KA4moAQLvTj-dX5a7HukYFah5cC5b1Xdk31ijbXgmxqFSxhCVyaxJ&host=tribes.sphinx.chat
```

**sphinx.chat** client makes a GET request to:

```
https://tribes.sphinx.chat/tribes/XtAHaiA0D1AvcHdA_AbE4W_DeM9NKGUWWMS_yRxJo_hhf3hm0jiu9k9RlGPoFJLBIvdRmPM1ZRxXvlaNUriPXT5F7HeX
```

and gets a JSON with the required information:

```json
{
  "uuid": "XtAHaiA0D1AvcHdA_AbE4W_DeM9NKGUWWMS_yRxJo_hhf3hm0jiu9k9RlGPoFJLBIvdRmPM1ZRxXvlaNUriPXT5F7HeX",
  "owner_pubkey": "03eab50cef61b9360bc24f0fd8a8cb3ebd7cec226d9f6fd39150594e0da8bd58a7",
  "owner_alias": "Upkiik Admin",
  "group_key": "MIIBCgKCAQEA9LE5RJf9rZ+Clb4UBv4odvCCfWdgvgDlx0js/V776koXxtkwSIIVrHyaLHsZ1phyFCe2S0Aem/qDUbVsxrSS0lWqkQTSpu0U1A8lugJNxBJ6kiTrLaev7Tg/zqWcPe3q12g+vgIzmJ+rbfYrJmpsF2jYbjdb45MsKmAqIg7nYMruYz5oGM2er3Qz+SrcepWuoKF7rDi1arHH9kU2AVepR8mA0mpbgyL8phKHQLyA6hWiQp1zxv5t510LhbcwE94TkWKzJTOHE+TwnOsuWPmgoIZ3eyMZGFjUdliA/mMeus+8F7VNo5PmI+2LxKwqDCR8AzwqXotOJoOQEAQ+5gzHSQIDAQAB",
  "name": "Upkiik",
  "description": "Sphinx Testing | Memes | News | Cryptocurrency \\u0026 Blockchain Discussion",
  "tags": [
    "Bitcoin",
    "Lightning",
    "Sphinx",
    "Crypto",
    "Tech",
    "Altcoins"
  ],
  "img": "https://memes.sphinx.chat/public/eWhdeUNo8MuAIZCiuHJFtSGHt08a5VAy3IIc7hM06Y8=",
  "price_to_join": 100,
  "price_per_message": 10,
  "escrow_amount": 0,
  "escrow_millis": 0,
  "created": "2020-05-28T18:48:11.605104Z",
  "updated": "2020-08-13T22:01:45.800488Z",
  "member_count": 35,
  "unlisted": false,
  "private": false,
  "deleted": false,
  "app_url": ""
}
```

---
NB: 
here
```
"escrow_amount": 0,
"escrow_millis": 0,
```
are, actually, message staking parameters, *staking amount* (in satoshis) and *staking timeout* (in milliseconds).

---

------------- ------------------------------------
#### :warning: 
here `"escrow_amount": 0, "escrow_millis": 0` are, actually, message staking parameters, *staking amount* (in satoshis) and *staking timeout* (in milliseconds).

----------------------------------------------------------------

|---|---|
|#### :warning: |here `"escrow_amount": 0, "escrow_millis": 0` are, actually, message staking parameters, *staking amount* (in satoshis) and *staking timeout* (in milliseconds).|

**sphinx.chat** client then sends a *Tribe join request* to the group owner pubkey as a special Sphinx message, over Lightning network.

### Tribe join request

User Relay sends a *keysend* message to the Admin Relay:

```ts
type: 14, chat: { members: [MY_PUBKEY]: {key: MY_ENCRYPTION_KEY, alias: MY_USERNAME } }
```
>> TODO: Check with Evan if it's the best possible representation of a Join request

*Keysend* must include amount of not less than *Price to join* of the group.

If *keysend* succeeds, users can safely assume that the group join has also succeeded, their **sphinx.chat** clients can connect to the specified **sphinx-tribes** server URL using MQTT protocol and read available group messages.

### Communication within a tribe

**sphinx-tribes** is an MQTT broker that any node can subscribe to. Message topics always have two parts: `{receiverPubKey}/{groupUUID}`. Only the owner of the group is allowed to publish to it: all messages from group members must be submitted to the owner as an LND keysend payment. the group `uuid` is timestamp signed by the owner.

![Tribes](https://github.com/stakwork/sphinx-tribes/raw/master/img/sphinx-tribes.png)

#### Message staking

To prevent spamming, group admins can enforce *Message staking*, that is requiring group members to include *staking amount*, in addition to (also optional) Group Message Price into each group message. The *staking amount* is to be returned automatically by the group admin Relay server after the expiration of *staking time* after the message publication in the group. If a member was expelled from the group, group admin keeps all the message stakes he had not returned to that moment.

#### Message price

To disincentivise spamming activity even more, and to collect some revenue on the group activity, group admin may enforce *message price* that will be collected by him- or herself.

#### Paid messages and media

As in private, one-on-one or *Private* group communication, price of a paid message or paid media will be collected by the message's author rather then group admin. *Message price* and *Staking amount* may still apply and will be collected by the group admin.

### Authentication

Authentication is handled by [sphinx-auth](https://github.com/stakwork/sphinx-auth)

## sphinx-tribes server deployment

### Prerequisites

* **docker**
* **golang**
* permanent public IP address (unless you are just testing)
* PostgreSQL, installed and [configured](#postgre)

### PostgreSQL configuration

* install the latest PostgreSQL (you can use normal, docker or remote deployment)
* create a user, e.g., `tribes_user`
* create a database, e.g., `tribes_db`, owned by `tribes_user`
* make sure PostgreSQL is being served on port 5432, unless you have configured a different port.

### Build

clone the Tribes repository:

```bash
$ git clone https://github.com/stakwork/sphinx-tribes.git
```

#### Build and run using **docker**

```bash
$ cd sphinx-tribes
$ docker build --no-cache -t sphinx-tribes .
$ docker run sphinx-tribes
```
>> TODO: Add traefik configuration (with Letsencrypt?)

#### Build and run as an ordinary Linux application

```bash
$ cd sphinx-tribes
$ go build
$ ./sphinx-tribes
```

You might also want to add an HTTPS proxy (e.g. NGINX) that would tunnel **sphinx-tribes** web UI over HTTPS on port 443, unless you want to use it only on your local network, etc.

## Connect your Relay server

To connect your Relay to your **sphinx-tribes** server put to Relay `config/app.json` 
```
tribes_host: <tribes server URL>:<web UI port>
```

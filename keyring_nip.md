# Nostr keyring nip 

This nip should propose an event for relatedkeys-relationships.

The keyring events, should have the following format:

Public keyring part, kind 17991 (publishes the needed public information for the related keys):

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the publisher>,
  "kind": 17991, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["P", <public key of a related key>],
    ...
    ["P", <public key of a related key>]
  ]
  "content": Stringified(
    [
      { pubkey: <32-bytes lowercase hex-encoded public key of key>, relation: "<S (for subkey) or M (for masterkey) or O (for otherkey)>", function: ["signing", "certify", "encryption", and/or "authentication""], delegation: "<optional delegation rules, such as defined in NIP-26, meaning a String with = (equal) / ! (not) / & (and) / | (or) / ( ) (flow) / < > (greater, smaller) / <> (not equal) and nostr event field or tag values, e.g. "kind=1|kind=2" (allowed to sign kind=1 or kind=2 events. If empty no restrictions.>" },
      ...,
      { ... }
    ]
  ),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

Private keyring part, kind 17992 (should mirror kind 17991 normally regarding the keys, but has additional private information for the keys):

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the publisher>,
  "kind": 17992, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["encryption", "e.g. nip44_v2, etc."]
  ],
  "content": as_in_tag_specified_encrypted(
    [
      { pubkey: <32-bytes lowercase hex-encoded public key of key>, seckey: <hex of secret key>, name: "<name of key/account>", description: "<optional desccription>" },
      ...,
      { ... }
    ]
  ),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

The related key should be included in the keyring event of the other related key respectively (e.g. masterkey includes subkey in its keyring event, subkey includes masterkey in its keyring event).
Saving the secretkey of the related key in the content-field is optional. E.g. it probably doesn't make sense to save the nsec of the masterkey in the subkey keyring event, or one doesn't want to save it for any related key, etc..
The private keyring part should be access-restricted by the relay (e.g. over AUTH-requirement).

## Nostr subkey login

Related to the keyring event here a protocol for conducting a login with a subkey from the keyring should be proposed. In the following are some possible subkey log in workflows.

### Option 1:

A masterkey keyring signer app creates a random secret key as subkey. It includes it in its keyring event as subkey.

It creates a bech32 string in the format 'nlogin...'. The bech32-string should follow in general the instructions of NIP19. The prefix should be 'nlogin'. TLV 0 is the 32 bytes of the subkey secret key. TLV 1 is the specified home-relays of the masterkey. TLV 2 is the pubkey of the masterkey. And TLV 3 is the keyring kind number 17991.

The bech32-string 'nlogin...' is then shared with the login-program, one wants to login with (login-program), e.g. via 'copy-and-paste' or a QR-Code, etc.. The login-program takes the string, decodes it and uses the login subkey and login relays to fetch, if already existing, the subkey-keyring event, and the masterkey-keyring event to verify the validity. If there is no subkey-keyring event yet, it creates a new one, including the masterkey reference.
The login-program can now use the subkey secret key and the user is logged in.

### Option 2:

User pastes into the login-program, it wants to log into, its NIP05 DNS-based identifier, or similar (e.g. nprofile-string). The login-program fetches the user-npub and the user-relays. The login-program creates a random subkey secret key and creates a 'nlogin...' bech32-string as defined in Option 1, where TLV 0 is the subkey secret key or subkey public key (if you don't want to share and save subkey secret key). TLV 1 is the relays from NIP05 DNS-based, or similar, identifier. TLV 2 is the npub from the NIP05 DNS-based, or similar, identifier. TLV3 is the keyring kind number 17991. 

Now the login-program shares the 'nlogin..' bech32-string with the masterkey signer app, e.g. via 'copy-and-paste' or a QR-Code, etc.. The masterkey signer app decodes the 'nlogin...' bech32-string. The masterkey signer app includes the subkey in its keyring event. The subkey keyring event is created by login-program. The login-program tries to fetch the masterkey-keyring event from the identifier relays to check the validity of login.

### Other option:

In general the informations subkey, masterkey and relays have to be transferred between the login-program and the masterkey signer app. All possible ways of transfer of these information are valid options.

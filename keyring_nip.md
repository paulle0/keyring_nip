# Nostr keyring nip 

This nip should propose an event for a keyring for subkey-relationships.
The relationship must only be of depth one. So a masterkey can have multiple subkeys. One subkey can have just one masterkey. And subkeys can have no further subkeys and masterkeys no further masterkeys.

The keyring events, should have the following format.

## Public keyring part, kind 17991

The public keyring part publishes the needed public information for the related keys.

For a masterkey, that publishes subkeys:
~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the publisher>,
  "kind": 17991, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["l", "rootkey"], 
    ["p", <public key of a subkey>], // masterkey can have several subkeys, and no masterkey, so all p-tags indicate subkeys
    ...
    ["p", <public key of a subkey>]
  ]
  "content": <for now empty, could be potentially used in future versions>,
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

For a subkey, that publishes relation to masterkey:
~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the publisher>,
  "kind": 17991, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["l", "subkey"],
    ["p", <public key of a masterkey>] // subkey must have just one masterkey, so just one p-tag
  ]
  "content": <for now empty, could be potentially used in future versions>,
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

## Private keyring part, kind 17992

The private keyring part should mirror the kind 17991 event regarding the keys, but has additional private information for the keys).

For a masterkey, that publishes subkeys:
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
      { pubkey: <public keys of the subkeys from 17991 event>, seckey: <optional secret key of the public key>, name: "<name of key/account>", description: "<optional desccription>" },
      ...,
      { ... }
    ]
  ),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

For a subkey, that publishes relation to masterkey:
No 17992-event.

Saving the secretkey of the related key in the content-field of the 17992-event is optional. E.g. you may want to distribute the subkey over several hardware via the masterkey or similar.
The private keyring part should be access-restricted by the relay (e.g. over AUTH-requirement).

## Nostr subkey login

Related to the keyring event here a protocol for conducting a login with a subkey from the keyring should be proposed. In the following are some possible subkey log in workflows.

### Example flow:

The keyring-program has the secret key of a subkey from the masterkey keyring. Either through the kind 17992 event, or through a nlogin in it itself. 

It creates a bech32 string in the format 'nlogin...'. The bech32-string should follow in general the instructions of NIP19. The prefix should be 'nlogin'. TLV 0 is the 32 bytes of the subkey secret key. TLV 1 is the specified home-relays of the masterkey. TLV 2 is the pubkey of the masterkey. And TLV 3 is the keyring kind number 17991.

The bech32-string 'nlogin...' is then shared with the program, with which one wants to login with (login-program), e.g. via 'copy-and-paste', a QR-Code, or a different way. The login-program takes the string, decodes it and uses the login subkey and login relays to fetch the kind 17991 events of the subkey and the masterkey and check its validity. If there is no subkey kind 17991 keyring event yet, it creates a new one, referencing the masterkey. If there is no, or a not matching kind 17991 event of the masterkey it throws an error.
If the events are valid, the login-program can now use the subkey secret key and the user is logged in.

### Other flow options:

In general the informations subkey, masterkey and relays have to be transferred between the login-program and the keyring-program. All possible ways of transfer of these information are valid options.

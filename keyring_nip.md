# Nostr keyring nip 

This nip should propose an event for otherkey/subkey/masterkey-relationships.

~~~
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized note data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the publisher>,
  "kind": 17991, // as defined in NIP-01 a replacable kind-number is used for this event-type
  "tags": [
    ["S", <32-bytes lowercase hex-encoded public key of a subkey>, <optional description / function (e.g. signing, certify, encryption, authentication) of the subkey>],
    ...
    ["S", <32-bytes lowercase hex-encoded public key of a subkey>, <optional description / function (e.g. signing, certify, encryption, authentication) of the subkey>],
    ["O", <32-bytes lowercase hex-encoded public key of an other key>, <optional description / function (e.g. signing, certify, encryption, authentication) of the other key>],
    ...
     ["O", <32-bytes lowercase hex-encoded public key of an other key>, <optional description / function (e.g. signing, certify, encryption, authentication) of the other key>],
    ["M", <32-bytes lowercase hex-encoded public key of a masterkey>, <optional description / function of the masterkey>],
    ...
    ["M", <32-bytes lowercase hex-encoded public key of a masterkey>, <optional description / function of the masterkey>]
  ]
  "content": nip44_encrypted("
    [
      { relation: "S or M or O from above", pubkey: <32-bytes lowercase hex-encoded public key of key>, name: "<name of key/account", description: "<optional desccription>", seckey: <hex of secret key>, function: "signing, certify, encryption or authentication" },
      ...,
      { ... }
    ]
  "),
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
~~~

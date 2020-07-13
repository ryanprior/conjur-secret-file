# Using Conjur & tar to store a secret file

This is based on the Conjur quick-start, so to get started you follow the [instructions](https://www.conjur.org/get-started/quick-start/oss-environment)

Then: 
1. load `policy/file-policy.yml` (as admin)
2. run `docker-compose exec client bash` to launch client shell
3. (in client shell) run `conjur variable values add secret-files/secret-script </data/secret-file.tar` to load the secret data
4. exit the client shell
5. (in host shell) run `docker-compose exec client conjur variable value secret-files/secret-script >round-trip.tar`

## Expected result

Here's where this repo, intended as a demonstration, turns into an issue report. I'd expect to get an identical tar file back.

## Actual result 

Instead, `diff secret-file.tar round-trip.tar` reveals that they are different, and hexdumping them shows there's some extra bytes in round-trip.tar which are screwing things up. I don't know where those bytes are coming from, so that's where I got lost.

```sh-session
ryan@swallowtail:~/dev/conjur-secret-file$ xxd secret-file.tar >f.hex
ryan@swallowtail:~/dev/conjur-secret-file$ xxd round-trip.tar >t.hex
ryan@swallowtail:~/dev/conjur-secret-file$ diff f.hex t.hex
34,39c34,39
< 00000210: 6173 680a 0a73 6574 202d 6575 0a0a 7365  ash..set -eu..se
< 00000220: 6372 6574 5f64 6174 613d 2263 6f72 7265  cret_data="corre
< 00000230: 6374 2068 6f72 7365 2062 6174 7465 7279  ct horse battery
< 00000240: 2073 7461 706c 6522 0a65 6368 6f20 2253   staple".echo "S
< 00000250: 6563 7265 7420 6461 7461 3a20 2473 6563  ecret data: $sec
< 00000260: 7265 745f 6461 7461 220a 0000 0000 0000  ret_data".......
---
> 00000210: 6173 680d 0a0d 0a73 6574 202d 6575 0d0a  ash....set -eu..
> 00000220: 0d0a 7365 6372 6574 5f64 6174 613d 2263  ..secret_data="c
> 00000230: 6f72 7265 6374 2068 6f72 7365 2062 6174  orrect horse bat
> 00000240: 7465 7279 2073 7461 706c 6522 0d0a 6563  tery staple"..ec
> 00000250: 686f 2022 5365 6372 6574 2064 6174 613a  ho "Secret data:
> 00000260: 2024 7365 6372 6574 5f64 6174 6122 0d0a   $secret_data"..
640a641
> 00002800: 0000 0000 0000                           ......
```

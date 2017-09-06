## Basic use

In the previous section we saw a part of the `identities` table, which stores the repo identities found, and their relationship with unique identities (`uuid`):

| id      | name                           | email                                | username | source | uuid    | 
|---------|--------------------------------|--------------------------------------|----------|--------|---------|
| 0cac4ef | Quan Zhou                      | quan@bitergia.com                    | NULL     | git    | 0cac4ef |
| 0ef1c4a | Jesus M. Gonzalez-Barahona     | jgbarah@gmail.com                    | NULL     | git    | 0ef1c4a |
| 11cc034 | quan                           | zhquan7@gmail.com                    | NULL     | git    | 11cc034 |
| 35c0421 | Alberto Martín                 | alberto.martin@bitergia.com          | NULL     | git    | 35c0421 |
| 37a8187 | Alberto Martín                 | albertinisg@users.noreply.github.com | NULL     | git    | 37a8187 |
| 3ca4e85 | Daniel Izquierdo Cortazar      | dicortazar@gmail.com                 | NULL     | git    | 3ca4e85 |
| 4fcec5a | dpose                          | dpose@sega.bitergia.net              | NULL     | git    | 4fcec5a |
| 5b358fc | dpose                          | dpose@bitergia.com                   | NULL     | git    | 5b358fc |
| 692ad15 | Andre Klapper                  | a9016009@gmx.de                      | NULL     | git    | 692ad15 |
| 6dcf98c | Daniel Izquierdo               | dizquierdo@bitergia.com              | NULL     | git    | 6dcf98c |
| 75fc28e | Santiago Dueñas                | sduenas@bitergia.com                 | NULL     | git    | 75fc28e |
| 7ad0031 | Alvaro del Castillo            | acs@thelma.cloud                     | NULL     | git    | 7ad0031 |
| 8fac15f | alpgarcia                      | alpgarcia@gmail.com                  | NULL     | git    | 8fac15f |
| 9aed245 | Alvaro del Castillo            | acs@bitergia.com                     | NULL     | git    | 9aed245 |

### Merging identities

It is obvious that there are some repo identities in it that correspond to the same person. In SortingHat, that means they should point to the same unique identity (`uuid`). But that is not the case, because in fact we have never told SortingHat they correspond to the same person. That operation is called "merging".

For example, let's merge repo identity `4fcec5a` (dpose, dpose@sega.bitergia.net) with `5b358fc` (dpose, dpose@bitergia.com), which I know correspond to the same person:

 ```bash
 (sh) $ sortinghat -u user -p XXX -d shdb merge \
   4fcec5a968246d8342e4acfceb9174531c8545c1 5b358fc11019cf2c03ea4c162009e89715e590dd
 Unique identity 4fcec5a968246d8342e4acfceb9174531c8545c1 merged on 5b358fc11019cf2c03ea4c162009e89715e590dd
 ``` 
 
Notice that we had to use the complete hashes (in the table above, and in the listing in the previous section, we shortened them just for readability). What we have done is to merge `4fcec5a` on `5b358fc`, and the result is:

```bash
$ mysql -u user -pXXX -e 'SELECT * FROM identities WHERE uuid LIKE "5b358fc%";' shdb
| id      | name  | email                   | username | source | uuid    |                                 
| 4fcec5a | dpose | dpose@sega.bitergia.net | NULL     | git    | 5b358fc |
| 5b358fc | dpose | dpose@bitergia.com      | NULL     | git    | 5b358fc |
```

The query looked for all rows in the `identities` table whose `uuid` field starts with `5b358fc`, and found two: the repo identifier `5b358fc`, which was already linked to it, but also `4fcec5a`, which we just merged onto it. This way we have learnt that when we merge, we merge a repo identity (the first argument) on a unique identity (the second argument).

We can follow this procedure for other identities that correspond to the same person: (Quan Zhou, quan@bitergia.com) and (quan, zhquan7@gmail.com); (Alberto Martín, alberto.martin@bitergia.com) and (Alberto Martín, albertinisg@users.noreply.github.com); and (Alvaro del Castillo, acs@thelma.cloud) and (Alvaro del Castillo, acs@bitergia.com):

```bash
(sh) $ sortinghat -u user -p XXX -d shdb merge \
  0cac4ef12631d5b0ef2fa27ef09729b45d7a68c1 11cc0348b60711cdee515286e394c961388230ab
Unique identity 0cac4ef12631d5b0ef2fa27ef09729b45d7a68c1 merged on 11cc0348b60711cdee515286e394c961388230ab
(sh) $ sortinghat -u user -p XXX -d shdb merge \
  35c0421704928bcbe3a0d9a4de1d79f9590ccaa9 37a8187909592a7b78559399105f6b5404af9e4e
Unique identity 35c0421704928bcbe3a0d9a4de1d79f9590ccaa9 merged on 37a8187909592a7b78559399105f6b5404af9e4e
(sh) $ sortinghat -u user -p XXX -d shdb merge \
  7ad0031fa2db40a5149f54dfc2ec2a355e9443cd 9aed245d9df109f8d00ca0e656121c3bdde46a2a
Unique identity 7ad0031fa2db40a5149f54dfc2ec2a355e9443cd merged on 9aed245d9df109f8d00ca0e656121c3bdde46a2a
``` 
  
### Showing information about a unique identity

Now, we can check how SortingHat is storing information about these merged identities, but instead of querying directly the database, we can just use `sortinghat`:

```bash
(sh) $ sortinghat -u user -p XXX -d shdb show \
  11cc0348b60711cdee515286e394c961388230ab
unique identity 11cc0348b60711cdee515286e394c961388230ab

Profile:
    * Name: quan
    * E-Mail: zhquan7@gmail.com
    * Bot: No
    * Country: -

Identities:
  0cac4ef12631d5b0ef2fa27ef09729b45d7a68c1	Quan Zhou	quan@bitergia.com	-	git
  11cc0348b60711cdee515286e394c961388230ab	quan	zhquan7@gmail.com	-	git

No enrollments
```

That is, we have two repo identities for this person, which we're identfying in the profile as `quan`, with email address `zhquan7@gmail.com`. Remember that the profile was already produced when each repo identity was added to the database, by creating a unique identity for it, and using the data in the repo identity for the profile.

We merged the repo identity (Quan Zhou, quan@bitergia.com) on the unique identity that corresponded to (quan, zhquan7@gmail.com), which had as profile (quan, zhquan7@gmail.com) as well. Therefore, the unique identity after the merge conserves the old profile, in this case (quan, zhquan7@gmail.com). Should we have merged the other way around, we would have consderved (Quan Zhou, quan@bitergia.com) as the profile, which in this case seems more convenient. So, we have to be careful about how to merge, if we want to conserve the most interesting profiles.

Unfortunately, we cannot redo the merge with the most convenient order:

```bash
(sh) $ sortinghat -u user -p XXX -d shdb merge \
  11cc0348b60711cdee515286e394c961388230ab 0cac4ef12631d5b0ef2fa27ef09729b45d7a68c1 
Error: 0cac4ef12631d5b0ef2fa27ef09729b45d7a68c1 not found in the registry
```

Why? Becauuse `0cac4ef` is no longer a valir unique identifier: we lost it when we merged the repo identity `0cac4ef`, which was the only one pointing to it. So, we cannot merge any repo identifier on it, since it no longer exists.

Later on we will revisit this case, since there are stuff that can be done: breaking the unique identifier into two. But for now, let's use another approach to solve this problem.

### Modifying profiles

We can just modify the profile for the unique identity, thus changing the profile for a person:

```bash
(sh) $ sortinghat -u user -p XXX -d shdb profile \
  --name "Quan Zhou" --email "quan@bitergia.com" \
  11cc0348b60711cdee515286e394c961388230ab
unique identity 11cc0348b60711cdee515286e394c961388230ab

Profile:
    * Name: Quan Zhou
    * E-Mail: quan@bitergia.com
    * Bot: No
    * Country: -
```

This way we have the name and email address we want. Using `--country` we can also set a country for the person.




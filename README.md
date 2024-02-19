# Daml Fundamentals Capstone Project: Digital Marble Racing

![Joe Mabel, CC BY-SA 3.0 <http://creativecommons.org/licenses/by-sa/3.0/>, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/a/a1/JM_marbles_01.jpg)

The project represents a very simple digital
[marble](https://en.wikipedia.org/wiki/Marble_(toy)) racing website where users
can be invited to races and win new marbles if they win.

## Project structure

### `daml/Main.daml`

The main logic for the project lives here. It contains the `Account`, `Race`,
and `MarbleWebsiteService` templates.

An `Account` represents the login of some user (although an user may have
multiple logins if they wish). The field it contains are the `admin : Party`,
for the website administrator, which is the signatory for many operations; the
`user : Party` to which this account maps to, its `userName : Text` which is the
`key` that identifies that login on the service, and all the
`marbles : [Marble]` which this account owns. By the default, the user gets 3
random marbles on its account creation.

A user can view another's profile, if its disclosed to another user, and the
admin can request that a new random marble is given to the user.

A `Race` represents a marble race, where the admin may add another account to
become a racer, abort the race, or start the race (if there are at least 3 users
on it). The winner of the race is chosen via the pseudo-random number generator,
and the winner will win a new random marble.

A `MarbleWebsiteService` is an interface for the user to interact with the
contract in general. For now, it only has an account creator, where the user may
choose a username for them.

So far, users need to pass a pseudo-random number generator around, but ideally,
in a real implementation, this would be implicit passed by the website and its
internals would not be disclosed.

### `daml/PseudoRandom.daml`

A simple
[linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator)
implementation, to generate random winners for races and random marbles.

### `daml/Setup.daml`

Contains modularized test setups, as well as `setupUsers` which is used to setup
the Navigator. A proposed workflow for the Navigator is detailed below.

### `daml/Test.daml`

Contains various happy and unhappy paths tests to check for the exposed choices.

## Proposed Navigator testing workflow

1. Login as `User1` (which is known as the `mainUser` in `Setup.daml`).
2. Click the `MarbleWebsiteService` contract.
3. Press `CreateAccount`, fill the details and press `Submit`. Suggested values:
    * `userName`: `h`.
    * `prng`:
        * `modulus`: `256`.
        * `multiplier`: `123`.
        * `increment`: `123`.
        * `seed:` `123`.
4. Logout.
5. Login as `Admin`.
6. Click the `Race` contract.
7. Press `AddToRace`, fill the details and press `Submit`. Suggested values:
    * `userName`: `h`.
    * `raceName`: `Navigator Racer`.
8. Return to the contracts page and find the new `Race` contract.
9. Press `StartRace`, fill the details and press `Submit`. Suggested values:
    * `prng`:
        * `modulus`: `5`.
        * `multiplier`: `321`.
        * `increment`: `321`.
        * `seed:` `321`.
10. With this, some user should have won (in my case it was `user3` as the time
of writing this README, which will appear as the topmost contract).

You may also search for templates and create new races, if you wish. This will
require adding at least 3 users, and starting the race. Keep in mind each time a
new user is added, a new `Race` contract is created, therefore you need to go
back to the user contracts list and update from there each time.

## Running

Use `daml start` to start the project on the Navigator.

## Testing

Use `daml test` to run all tests.

## Challenges

* A lot of issues and design decisions implementing this project arose from the
UX of using the Navigator. Most notably, in the Navigator, there is no
equivalent of `submitMulti`, which required moving a lot of choices around and
changing their design. For example, previously I would have liked to have the
user join a race, not the admin invite them, but that would require disclosure
to the participants and the usage of `submitMulti`, or even sometimes trying to
implement a proposal-accept pattern that would also have required `submitMulti`.
* A lot of the design decisions and time spent were also due to not
understanding that well how the privacy models of Daml work, so sometimes I
could not understand why some lookup failed (and how to remedy), or why
permission from some party was missing (and how to provide it). But later, I
understood my project proposal doesn't care so much about privacy, so Canton is
also not the best for what I originally wanted. However, I think what I got to
now is good enough.
* I do not use conditionals (`if _ then _ else _`), but I use pattern matching
(`case _ of _`) and guards (`f | _ -> _`), which I hope is enough for the
grading of the project.
* I could not find a nicer way to easily pass around a PRNG without needing to
manually type it in the Navigator. I wish there was a function to convert the
current time to Unix time (`Int`), so at least I could change the seed based on
time.
* I created the project with the empty template, but I removed
`--wall-clock-time` from the `daml.yaml` file.
* The language server would sometimes crash, or misbehave in the presence of
errors, which made development more difficult. Sometimes the information
provided by hovers was not enough, so using the documentation was a must.
* Moreover, the error messages could be improved. I frequently had to deal with
confusing messages about ambiguous variables coming from template and choice
fields. Thankfully, those resulted from mistakes and could be easily fixed.
* Lack of more utilities in the standard library, e.g., a function that combines
`traverse` and `foldl`. But `foldl` was enough to implement what I needed.

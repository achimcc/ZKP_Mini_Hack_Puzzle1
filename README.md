## zkhack-there-is-something-in-the-AIR

**DO NOT FORK THE REPOSITORY, AS IT WILL MAKE YOUR SOLUTION PUBLIC. INSTEAD, CLONE IT AND ADD A NEW REMOTE TO A PRIVATE REPOSITORY, OR SUBMIT A GIST**

# Trying it out

Use `cargo run --release` to see it in action

# Submitting a solution

[Submit a solution](https://xng1lsio92y.typeform.com/to/aUSCQrcW)

# Puzzle description

```
    ______ _   __  _   _            _
    |___  /| | / / | | | |          | |
       / / | |/ /  | |_| | __ _  ___| | __
      / /  |    \  |  _  |/ _` |/ __| |/ /
    ./ /___| |\  \ | | | | (_| | (__|   <
    \_____/\_| \_/ \_| |_/\__,_|\___|_|\_\

Alice implemented a Semaphore protocol to collect anonymous votes from her friends on various
topics. She collected public keys from 7 of her friends, and together with her public key, built
an access set out of them.

During one of the votes, Alice collected 9 valid signals on the same topic. But that should not be
possible! The semaphore protocol guarantees that every user can vote only once on a given topic.
Someone must have figured out how to create multiple signals on the same topic.

Below is a transcript for generating a valid signal on a topic using your private key. Can you
figure out how to create a valid signal with a different nullifier on the same topic?
```

# Puzzle Solution

The attack vector of the Semaphore protocol arises from the lack of checking the full set of required boundary constraints of the nullifier section of the execution trace. These are the values represented by the indices 13..23. Here we don't impose the required boundary constraints of `state[13]==FELT::ZERO`, `state[14]==FELT::ZERO` and `state[15]==FELT::ZERO`.

To generate new votes from an exisiting private key, we can't modify the private key, since the merkle tree implementation part of the Air is correct. But we can modify the initial value of e.g. `state[13]`. I do this here by replacing its initial value with `state[13]=Felt::new(1)`. Now, the execution trace yields a different Nullifier, and the proof verification will initially fail. But instead of computing the Nullifier with `hash(priv_key, hash(topic))`, I can simply print out what the Prover computes as Nullifier after one successfull hash cycle.

Now I can use this Nullifier as input value for the Signal, and the corresponding Air constrains are fulfilled!

To make the Semaphore protocol resistant against this attack, we could add additional boundary conditions like:

```rust
            Assertion::single(13, 0, Felt::ZERO),
            Assertion::single(14, 0, Felt::ZERO),
            Assertion::single(15, 0, Felt::ZERO),
```

after line 134 of `src/air/mod.rs`.

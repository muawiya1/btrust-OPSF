# **Learning Bitcoin Core Through Testing and PR Review**

Exploring Bitcoin Core has been an eye-opener for me. Every day I learn something new, and it feels great t~~o~~ run a command and get the result I expected. Going further into the codebase has introduced me to another part of that experience: understanding why a command fails and how a proposed change addresses the problem.

Over the past few weeks, I have been exploring Bitcoin Core’s tests and reviewing merged pull requests. One wallet fix was particularly useful because I could connect the reported error to the implementation, examine the added test, and compare behaviour before and after the change.

As someone still finding his way around the project, this gave me a manageable way to approach a review. I had one problem to understand and a specific result to investigate.

## **Preparing to Test the Changes**

To start with, I ran the unit tests using `ctest --test-dir build --output-on-failure`, then explored the functional tests through `build/test/functional/test_runner.py`.

Some results required attention before I could continue. The `script_assets_tests` unit test was skipped because `DIR_UNIT_TEST_DATA` was not set. Its required file, `script_assets_test.json`, was available separately in the `bitcoin-core/qa-assets` repository. After downloading the file and setting its location, I reran the test successfully.

<img width="857" height="174" alt="Screenshot 2026-10-01 at 22 18 27" src="https://github.com/user-attachments/assets/088c8655-a0ea-408d-8b2c-a4c37f520710" />

*Figure1: Showing the skipped test*

<img width="1709" height="290" alt="Screenshot 2026-10-01 at 22 17 36" src="https://github.com/user-attachments/assets/b7c84e40-615c-4a5c-8328-ea538f0ba1a6" />

*Figure2: Showing the skipped test passed after fix*

Two functional tests also failed because they were using my system’s default Python 3.9.6. I had a newer version installed through Homebrew, so I reran them using `/opt/homebrew/bin/python3`, and both passed.

<img width="824" height="469" alt="Screenshot 2026-10-01 at 22 40 20" src="https://github.com/user-attachments/assets/ea5e6c8f-1270-4366-aab5-f074dc1602b6" />

*Figure3: Showing two failed functional tests*

<img width="814" height="458" alt="Screenshot 2026-10-01 at 22 45 25" src="https://github.com/user-attachments/assets/1516447d-7a05-48a6-b8ee-22b781672c89" />

*Figure4: Showing the two failed tests passed the functional test after the fix*

These problems became useful lessons when starting to review changes. Before treating a failed test as evidence of a bug, you need to check what caused the failure. Missing data or an unsupported interpreter could prevent the test from reaching the behaviour I wanted to examine.

## **Understanding the Problem Before Reading the Fix**

The PR of focus here for this article was [wallet: accept uppercase addresses without amount in sendall- \#36375](https://github.com/bitcoin/bitcoin/pull/36375), which addressed a problem with valid uppercase Bech32 addresses supplied without an amount to `sendall`.

In the test scenario, the recipient should receive the wallet’s balance minus fees. The uppercase form represented a valid destination, but the command returned a “below dust threshold” error.

This raised a question for me: how did the way an address was written affect the amount assigned to its output?

Understanding the expected result gave me a starting point for reading the diff. I needed to follow how `sendall` remembered recipients without amounts and how it later identified their outputs when distributing the remaining funds.

## **Following the Amount Through the Code**

In [`src/wallet/rpc/spend.cpp`](https://github.com/bitcoin/bitcoin/blob/master/src/wallet/rpc/spend.cpp), the original implementation stored addresses supplied without amounts in a `std::set<std::string>`.

These recipients initially received placeholder output amounts of zero. Later, the code went through the transaction outputs to identify which ones should receive a calculated share of the remaining amount.

For each output, it obtained the destination from `out.scriptPubKey` using `ExtractDestination()`. It then called `EncodeDestination()` to turn that destination back into an address string and checked whether the string was present in the stored set.

This was where the mismatch occurred. For Bech32 addresses, `EncodeDestination()` returned the lowercase form, while the set contained the uppercase text supplied by the user.

The lookup therefore failed. The output did not enter the branch that assigned its calculated amount, and its value remained zero. The later dust check rejected it.

Following this sequence made the error easier to understand. The dust check was responding to the zero-valued output it received. The earlier lookup explained how that output had reached the check without its amount being assigned.

## **Examining Whether the Change Addressed the Cause**

The main change replaced the collection’s element type:

<img width="615" height="273" alt="Screenshot 2026-10-07 at 11 01 50" src="https://github.com/user-attachments/assets/04bf1db0-8537-4e37-accd-1185e334a4d1" />

*Figure5: Showing the changes in the spend.cpp file*

When processing a recipient, the updated code used `DecodeDestination()` to store the destination represented by the address. Later, after extracting a destination from an output’s script, it checked that destination directly against the set.

<img width="752" height="426" alt="Wallet_diff_changes_sendall" src="https://github.com/user-attachments/assets/2727a312-d5b3-4f01-8acf-83db3895e69d" />

*Figure6: Showing the updated changes in the spend.cpp file*

What mattered to me was how these changes worked together. Changing the collection alone would not explain the fix; the stored value and the value used for the lookup also had to represent the same thing.

Previously, the lookup depended on matching two address strings produced in different ways. With the change, both sides used `CTxDestination`, so a difference in the address’s letter case no longer prevented the output from being recognised.

This also kept the fix close to the cause of the problem. It corrected the identification of outputs that needed an amount assigned. The dust check could continue checking the resulting amounts.

Reading the surrounding code, rather than only the changed lines, helped me connect the new comparison with the branch it allowed the output to enter.

## **Reading the Test’s Expectations**

After examining the implementation, I looked at `sendall_uppercase_address()`, the test added to `wallet_sendall.py`.

The test exercised the specific condition described in the PR, supplying an uppercase address without an explicit amount. It then checked the result of the operation, including that the recipient received the wallet balance minus fees and that the sending wallet was completely swept.

These assertions were important because a successful RPC response alone would not establish that the amount had been distributed correctly. The reported bug involved an output retaining the wrong amount, so the test needed to examine the financial result of the operation.

Here, I began to see how the implementation and test explained each other. The code change corrected the destination lookup, while the assertions checked the outcome that lookup was supposed to enable.

## **Comparing the Behaviour Before and After**

I ran the updated `wallet_sendall.py` against the version before the fix. It reached the uppercase-address scenario and failed with the reported dust error, returning exit code 1\.

<img width="748" height="550" alt="wallet_before_fix" src="https://github.com/user-attachments/assets/80c41755-8141-480b-a27b-01a7e95185b6" />

*Figure7: Showing the test failed with “below dust threshold error” before the fix*

I then ran the same test against the version containing the change. It passed with exit code 0\.

<img width="744" height="240" alt="wallet_after_fix" src="https://github.com/user-attachments/assets/a62cf91c-949e-414f-9142-6aea439fb59f" />

*Figure8: Showing the test passed after the fix*

The reason for the earlier failure mattered as much as the failure itself. It matched the behaviour described by the PR, giving me evidence that the test exercised the problem under investigation. A failure caused by the Python version or missing test data would not have demonstrated that.

Using the updated test against both versions also meant that I was comparing them against the same expectation. The old implementation failed the added scenario, while the changed implementation satisfied its checks.

This gave me a clearer understanding of regression testing. The added test preserved a concrete example of the bug and checked the expected behaviour, making it useful for detecting a future change that brought the problem back.

## **What the Review Taught Me**

The most useful part of this exercise was connecting the different pieces of the review. The PR description explained the symptom. The surrounding code showed how the amount remained zero. The diff changed the comparison responsible for that behaviour, and the test checked the resulting transfer.

Together, these gave me a stronger explanation than simply saying that the test passed.

I am still learning how to assess the wider effects of a change. My local results demonstrated the reported scenario and the checks exercised by the test; they did not establish that every possible `sendall` case was correct.

For me, that is also part of learning to review: being able to explain both the evidence and its limits. Following one small fix closely gave me more confidence in reading Bitcoin Core, because I could describe what changed, why it mattered, and how I checked the result.

## **References and Further Reading**

* [Bitcoin Core repository](https://github.com/bitcoin/bitcoin): the source code and documentation for exploring the project.  
* [PR \#36375](https://github.com/bitcoin/bitcoin/pull/36375): the uppercase-address problem in `sendall`, its fix, and the review discussion.  
* [Code changes and regression test](https://github.com/bitcoin/bitcoin/pull/36375/commits/df2994b14a4ba26c4ccebd6df4c24d8aaf22d851): the destination comparison changes and the test added to check the corrected behaviour.  
* [Unit-test documentation](https://github.com/bitcoin/bitcoin/blob/master/src/test/README.md): how to run unit tests and select individual test suites or cases.  
* [Functional testing instructions](https://github.com/bitcoin/bitcoin/blob/master/test/README.md): how to use the test runner, run selected tests, and investigate failures.  
* [External unit-test data](https://github.com/bitcoin-core/qa-assets/tree/main/unit_test_data): the location of `script_assets_test.json`, which I needed to run the skipped test.  
* [Contributing guide](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md): the contribution process, expectations for testing and peer review, and the meaning of review terms such as ACK and NACK.

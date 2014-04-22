Testing the coin control feature of Peerunity
===================================================

This report covers test results of the coin control feature of Peerunity v0.4.0 released [here](http://www.peercointalk.org/index.php?topic=2648.msg23426#msg23426). The test method used is basically clicking/typing on the GUI of Peerunity running on Peercoin testnet, and comparing the results with those specified in [_Yet another Coin Control Release_ by cozz](https://bitcointalk.org/index.php?topic=144331.0). The tests are done on windows 7 professional 32bit. Peerunity distribution peerunity-rc.zip is downloaded and unzip'ped in D:\bin

There are three addressed used in the test 

   | name | address | initial balance |
   | ----- | --------- | ---------------- |
   | test1 | mkLh7wYGYQiKVm8tLwCs9LusKF2y1gffm9 | 8.01PPC |
   | test2 | n2LzA1UH46QZPdCGvN2NorPeqg1BbTuw4g | 2PPC |
   | test3 | mpRAV4VSsxmKh5nRpDuQPquhdHPKu4xUuS | 7PPC |

### start peerunity

* Run peerunity with a bat file that has the command
```
D:\bin\peerunity-rc\peerunity-rc-qt -conf=D:\bin\peerunity-rc\ppcoin.conf
```
The config file D:\bin\peerunity-rc\ppcoin.conf contains
```
datadir=D:\bin\peerunity-rc
testnet=1
server=1
rpcuser=mh
rpcpassword=x
daemon=1
genproclimit=1
daemon=1
```
(Note for future upgrade: with 0.9.1 btc wallet, datadir must be specified in the command line, not in the config file)

* peerunity-rc is started. In the debug window, Information tab, "On testnet" box is in checked state. 
* peerunit-rc successfully syncs with the testnet. "D:\bin\peerunity-rc\testnet" subdirectory is created with downloaded block chain.

### start coin control

* In settings->options->display check "Display coin control features (expert only!)".
* The following GUI functions are verified (I did not check if custom change address works)

> _Main_
>   * Settings checkbox "Display coin control features (experts only!)" (default=no)
>
> _Tab Send coins_
>   * Button Inputs
>
>     ..* click on this button opens actual coin control dialog. If no Inputs are selected "automatically selected" is shown.
>   * Change Checkbox
>     ..* checked -> provide custom change address


* Send 3 PPC from faucet2 to faucet1 to 1) create change 2) to confirm if in coin control, if no boxes
[code]are selected -> coin control inactive (just as normal)[/code]
txid 8c1a65708126efb9c48535a4a1db049684745e78b72dd0f095e34151e9a346db

In [i]Coin Control Dialog[/i] The following are confirmed
[quote]
[list]
[li]Shows a list of all unspent outputs with two view modes
[list]
    [li]tree mode: outputs including change are grouped by wallet address[/li][/list]
        tree can be opened showing the actual outputs for this wallet address including change
        if change, the change bitcoin address is shown in column "address", otherwise the column "address" is empty, because its a direct output of the wallet address having the bitcoin address already shown in the parent node (same with label)
    [list][li]list mode: simple list of all unspent outputs[/li]
[/list]
[/li][/list]
[/quote]

* Select faucet3 address in coin control and send 2PPC to faucet2. Examine the results in coin control confirming 
[quote]
[list]
[li]select outputs by checkbox -> only the checked outputs are used when sending a transaction[/li][/list]
[/quote]
(a strict confirmation should come from a block explorer) txid 04494cd36def0b679dc264d8cf359089fcb38f9a32b8d0fc218f7fca57cc4901

* Confirm functions of the following
[quote]
[list]
[li]check/uncheck all by clicking on "(Un)select all"[/li]
[li]sort colums[/li]
[li]tooltip available in column list mode in column label for the change (shows from which address the change came from)[/li]
[/list]
[/quote]
Note for the last item: the tooltip not only shows in list mode but also in tree mode.

* The following are verified [color=red]except I do not see "lock" and "unlock"[/color]
[quote]
[list]
[li]Context menu
[list]Copy to clipboard (amount,label,address,transaction id,lock,unlock)[/list]
[/li]
[/list]
[/quote]

These labels are tested [color=blue]except I cannot verify priorities because I do not know how priority levels are assigned[/color]. 
[quote]
[list]
[li]Labels at the top
[list]Quantity: number of selected outputs
Amount: sum of selected unspent outputs
Fee:   see "Calculation of fee and transaction size" below
minus fee: simply the amount shown is "Selected" minus the amount shown in "Fee"
Bytes: see "Calculation of fee and transaction size" below
Priority: priority = coinage / transactionsize. coinage = value * confirmations.  miners order transactions by priority when selecting which go into a block
Low Output: "yes" if any recipient receives an amount < 0.01BTC
Change: shows the change you get back
[/list]
[/li]
[/list]
[/quote]
Low Output is yes when the amount is less than 0.01 PPC.

* [color=red]I cannot copy amount to clipboard by direct right clicking the labels[/color]
[quote]
[list]
[li]direct right click the labels for copy amount to clipboard
[/li]
[/list]
[/quote]

The following are verified except that "The amount exceeds your balance" is "Insufficient funds!"
[quote]
[b]Selection[/b]
In this version of coin control, all selected outputs are going into the transaction for sure!!
Of course, if you select more than you actually send, the bitcoin core will send the rest back to you as change, just as normal.
And of course, if you select less than you send you will get "The amount exceeds your balance".
And as already mentioned, if none are selected, coin control is inactive, this means everything is just the same as without coin control.

[b]Fee[/b]
If the sum of selected outputs minus the amount you are going to send is smaller than the required fee, you will probably get
"The total exceeds your balance when the transaction fee is included"
This is because you didnt select enough outputs to pay the fee.
You always must select enough outputs, so that those outputs can pay the fee.
[/quote]

* [color=red]I do not know how to independently calculate transaction size so I didn't test transaction size calculation[/color]
* Peercoin has a fixed transaction fee so the part about fee and free transaction calculation shouldn't apply.
[quote]
[b]Calculation of fee and transaction size[/b]
The fee is calculated according to the fee set in the Settings menu.
The calculation assumes 2 outputs in total. One for the destination address and one for the change.
The formula is nBytesOutputs + (2 * 34) + 10. nBytesOutputs is the sum of selected outputs, 148 or 180 bytes per output, depending if compressed public key.
Due to the inner workings of bitcoin the size per output is actually +/- 1 byte. Meaning the shown calculation is not always 100% correct.
If you send exactly "selected minus fee" then you will not have change (1 output only). The transaction will then be 34 bytes smaller as what was calculated before.

[b]Free Transactions[/b]
In order to be able to send a free transaction, you need to follow the rules:
     - transaction size must be < 10000 bytes
     - priority must be at least "medium"
     - any recipient must receive at least 0.01BTC
     - change must be either zero or at least 0.01BTC
  If you violate one rule you will see a min-fee and also the labels turn red:
  Bytes.Priority,Low Output,Change. Depending which rule you violated.
  Those 4 labels also have tool tips explaining this.
  Also remember that violating one of the first 2 rules means 0.0005 PER kilobyte min-fee,
  while violating one of the last 2 means 0.0005 min-fee only.
[/quote]

[b]Summary[/b]
The coin control implmented in peerunity_coin-control passed tests of all main function items [url=https://bitcointalk.org/index.php?topic=144331.0]specified by cozz[/url]. Three minor items did not pass test and are marked in red above. One untested item is marked in blue.

Good job glv!


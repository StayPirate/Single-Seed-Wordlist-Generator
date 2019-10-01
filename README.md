# Single Seed Wordlist Generator

Password cracking is one of the most important phases during red team operations and using the right wordlists could really make the difference. Aimed custom wordlists are easy to generate since internet is plenty of tools, but what about password based on a specific word like the name of the company.

### Why these rule-files
In my experience I noticed that many users, especially sysadmins, derive their passwords from the name of the company they work for. Some of these passwords are allowed by the company's password policy since containing numbers, uppercase letters, lowercase letters and are long enough. Unfortunately for them (but not for us) these passwords are easy to bruteforce because are based on a low entropy seed: the name of the company.
### Hashcat's rules to the rescue
There are currently no tools on internet aimed to generate an exhaustive wordlist starting from a single word. Hence, I thought to use [hashcat](https://hashcat.net/wiki/doku.php?id=hashcat) and [maskprocessor](https://hashcat.net/wiki/doku.php?id=maskprocessor) to achieve this goal. The secret was (again) _divide et impera_. Starting from oclHashcat-plus v0.07 [multiple rule-files](https://hashcat.net/wiki/doku.php?id=rule_based_attack#multi-rules) can be passed to hashcat and these are not executed in a sequence, instead each rule of each rule-file is combined with each rule of each rule-file! In order to bring the output of each rule-file to the hashcat's output (and not only the output of the last rule-file declared in the command-line) I added a simple _do-nothing rule (:)_ as the first rule of each rule-files. In this way, the output of a rule-file which flows as input of the next rule-file will be printed out untouched as part of the second rule-file's output.

### How to use it
> First of all, ensure you have [hashcat installed](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#installation) and configured.

Let's pretend we are attempting to crack dumped hashes from a company called "_Example_". To generate the wordlist with passwords based on the word _Example_ we run hashcat with the following options:
```    
echo example | hashcat -r 01-mangle.rule -r 02-case_toggle.rule -r 03-l33t/l33t_micro_1234.rule -r 04-prefix.rule -r 05-suffix.rule --stdout | sort -u > example_wordlist.txt
```
It generates `example_wordlist.txt` (~100Mb) with **9.6 millions** unique words based on the word _example_. Consider [`rockyou.txt`](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) contains 14.3 millions unique passwords.

### ⚠️ Beware!
**Hashcat generates in-memory wordlists**. This means that in order to generate a huge wordlist you need enough free central memory (RAM) to store it during its generation. If hashcat fails, try to use fewer rule-files or rule-files with fewer rules.  
Read [*Rule-files*](#Rule-files) paragraph below to understand what the rule-files in this repository are made for.

### Rule-files
Exploiting my Roman ancestors' [idea](https://en.wikipedia.org/wiki/Divide_and_rule): I created different rule-files, each one with different string manipulation purposes.

 * [01-mangle.rule](01-mangle.rule) is used to mangle words. 
   * Swapping adjacent letters: _examlpe_. (not in the rule-files)
   * Removing vowels: _xmpl_.
   > In case the company name starts with a vowel you can add here a rule
      to remove all vowels except for the first letter of the name: **exmpl** instead of **xmpl**.
   * Keep only the first three letters: _exa_.
  
 * [02-case_toggle.rule](02-case_toggle.rule) as the name suggests, is used to change the case of letters.
   * Capitalize the first letter and lower the rest.
   * Uppercase all letters.
  
 * [03-l33t](03-l33t) This is the most complicated set of rules. Base on how you use it, it can drastically change the size of your wordlist. Check out the README on this folder for a better overview of these rule-files.
 Moreover, on internet I haven't found any good rule-file for l33t encoding. These rule-files can easily be used to l33t-encode any other wordlists, feel free to use them as you prefer.
 
 * [04-prefix.rule](04-prefix.rule) prepends one or more chars to the word.
   * Prepend a single symbol to the word: _**?**example_.
   * Prepend one or more digit: _**123**exaple_. (not in the rule-files)
  
 * [05-suffix.rule](05-suffix.rule) append one or more chars to the word.
   * Single digit: _example**1**_.
   * Double digit: _example**12**_.
   * Single symbol: _example**?**_.
   * Single symbol + single digit: _example**?1**_.
   * Single digit + single symbol: _example**1?**_.
   * Single symbol + double digit: _example**?11**_.
   * Double digit + single symbol: _example**12?**_.
   * Year (1900-2029): _example**2010**_.
   * Single symbol + year (1900-2029): _example**-2010**_.

 * [06-case-toggle-multiple-words.rule](06-case-toggle-multiple-words.rule) aka Camel Case.
   > In case the name of the company is composed of more than one word.
  
   * Lower case the whole line, then upper case the first letter and every letter after a space: _Company Name_.
  
### Order matters
Based on which order you pass the rule-files to the command-line (-r options) you would get different wordlists. Always consider the output of one file-rule will be used as input for the next one, like in a pipeline.  
For instance, let's take the word _Company_ and the following two file-rules. 
  
file-rule1:
```
:
$A
```
file-rule2
```
:
sA4
```

if we apply file-rule1 first, then file-rules2 we get three unique words:
```
echo Company | hashcat -r rule-file1 -r rule-file2 --stdout | sort -u
Company
CompanyA
Company4
```
But if we apply file-rule2 first and file-rule1 for second, we get two unique words:
```
echo Company | hashcat -r rule-file2 -r rule-file1 --stdout | sort -u
Company
CompanyA
```
That's the reason why I named my rule-files with number-prefix, to easily choose the best order when I use them.

### Tune for best results
To achieve the best result I recommend to fork this repo, then tune/customize rule-files according to your seed (the word you are going to use / the company's name). The Hashcat's [official docs](https://hashcat.net/wiki/doku.php?id=rule_based_attack) is a great start point, rules are well documented there and [maskprocessor](https://hashcat.net/wiki/doku.php?id=maskprocessor) is your best friend. Moreover, take a look at comments on rule-files on this repo.

### Share your custom rule-files
If you create some cool rules which increase your successful crack-ratio, please submit it via PR. Thank you!

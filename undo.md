# PicoCTF platform: Undo

The challenge involves connecting to a remote machine and reversing multiple text transformation to recover the original flag.

## Goal
At each step the machine presents transformed text and asks for the correct command needed to reverse the transformation.

## Solution
1. Decode the Base64 string
Use the `base64` command with `-d` flag to decode the base64-encoded text.
```bash
echo "KTJxNW85NjQ1LWZhMDFnQHplMHNmYTRlRy1nazNnLXRhMWZlcmlyRShTR1BicHZj" | base64 -d
)2q5o9645-fa01g@ze0sfa4eG-gk3g-ta1ferirE(SGPbpvc
```
2. Reverse the text
Use the `rev` command to reverse the order of characters in the string.
```bash
echo ")2q5o9645-fa01g@ze0sfa4eG-gk3g-ta1ferirE(SGPbpvc" | rev
cvpbPGS(Eriref1at-g3kg-Ge4afs0ez@g10af-5469o5q2)
```
3. Replaces dashes with underscores
Use the `tr` command to translate characters. the hint indicates that underscores (_) were replaced with dashes (-), so we convert them back.
```bash
echo "cvpbPGS(Eriref1at-g3kg-Ge4afs0ez@g10af-5469o5q2)" | tr '-' '_'
cvpbPGS(Eriref1at_g3kg_Ge4afs0ez@g10af_5469o5q2)
```
4. Replace parentheses with curly brackets
Use `tr` again to replace parentheses with curly brackets.
```bash
echo "cvpbPGS(Eriref1at_g3kg_Ge4afs0ez@g10af_5469o5q2)" | tr '()' '{}'
cvpbPGS{Eriref1at_g3kg_Ge4afs0ez@g10af_5469o5q2}
```
5. Decode the ROT13 transformation
ROT13 is a Caesar cipher that shifts each letter with 13 positions ahead in the alphabet.  
Use `tr` to reverse the ROT13 text.
```bash
echo "cvpbPGS{Eriref1at_g3kg_Ge4afs0ez@g10af_5469o5q2}" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
picoCTF{Revers1ng_t3xt_Tr4nsf0rm@t10ns_5469b5d2}
```

### Final Flag:
picoCTF{Revers1ng_t3xt_Tr4nsf0rm@t10ns_5469b5d2}

## Commands Used
* base64 -d # decodes base 64 string
* rev # reverses order of characters in string
* tr '-' '_' # replaces dashes with underscores
* tr '()' '{}' # replace parentheses with curly brackets
* tr 'A-Za-z' 'N-ZA-Mn-za-m' # decodes/encodes rot13 cipher
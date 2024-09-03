# data-mod

This directory has a modified form of PPCHY created for input to NLP applications, such as a part-of-speech tagger and syntactic parser.  It was created using the code at [ppchyprep](https://github.com/skulick/ppchyprep.git). The main goals are:

1. Conversion of the romanized form to the Yiddish script:  This makes use of Isaac Bleaman's [yiddish package](https://github.com/ibleaman/yiddish), which in turn required modification of the romanized form in some cases, and a misc. number of changes detailed at ppchyprep.  For further discussion of this conversion issue, see [A Part-of-Speech Tagger for Yiddish](https://arxiv.org/abs/2204.01175).
2. Consistency with the original source text:  words that are marked in the treebank as having been split are joined together, and in cases where they weren't marked as such (contractions), a variety of heuristics were used to do.  Conversely, words that were merged with an underscore in PPCHY are separated in this version (in consultation with Beatrice as to the best way to do this).  The reasons for goals 1 and 2 is that NLP applications will work on Yiddish source text, not (with maybe some exceptions) romanized forms, and so we want the annotated text that the pos-tagger or parser trains on to be as close as possible to the texts to be tagged and parsed.
3. Ease of use for the NLP applications.  The treebank leaves are sometimes split from whitespace-delimited words as encountered in source texts, a common situation, and we have adopted the CONLL format for listing such cases in tabular format (in the `pos` directory).  These listings include both the romanized and Yiddish script forms of the words.  Since the trees and these lexical items need to be used together by the NLP software, there is also a `json` directory that includes all the information together. The NLP software can then create versions of the trees with the Yiddish script forms substituted for the romanized words.

# Directories

The data directory is unchanged from the PPCHY repository.  There is a sister directory `data-mod` with four subdirectories:

1. psd: These include the misc changes to the trees, and also all cases of `*` and `0` have been changed to have a `-NONE-` parent, in keeping with the Penn Treebank style.  e.g.

```
( (IP-MAT (INTJ nu)
	  (PUNC ,)
	  (VBF iz)
	  (NEG nisht@)
	  (ADVP (ADV @o))
	  (NP-SBJ (Q vos)
		  (CP-EOP (WNP-1 0)
			  (IP-INF (NP-ACC *T*-1)
				  (TO tsu)
				  (VB redn))))
	  (PUNC .))
  (ID 1910E-GRINE-FELDER,106.1753))
```

is changed to

```
( (IP-MAT (INTJ nu)
	  (PUNC ,)
	  (VBF iz)
	  (NEG nisht@)
	  (ADVP (ADV @o))
	  (NP-SBJ (Q vos)
		  (CP-EOP (WNP-1 (-NONE- 0))
			  (IP-INF (NP-ACC (-NONE- *T*-1))
				  (TO tsu)
				  (VB redn))))
	  (PUNC .))
  (ID 1910E-GRINE-FELDER,106.1753))
```

Similary, a `-NONE-` is inserted into all `CODE`s that are internal to a tree, so that e.g. `(CODE {GLOSS:dowry})`  becomes `(CODE (-NONE- {GLOSS:dowry}))`.  (This made it easier for the conversion code to keep track of which are the real leaves.)

2. pos: A listing of the lexical items in each tree.  e.g. for the above:

```
SENT	1910E-GRINE-FELDER,106.1753
1	nu	INTJ	נו	nv	_
2	,	PUNC	,	,	_
3	iz	VBF	איז	Ayz	_
4-5	nishto	NEG~ADV	נישטאָ	ny$tAo	_
4	nisht	NEG	נישט	ny$t	_
5	o	ADV	אָ	Ao	_
6	vos	Q	װאָס	VAos	_
7	tsu	TO	צו	qv	_
8	redn	VB	רעדן	redN	_
9	.	PUNC	.	.	_
```

Note that the word that was split in the tree (nishto) is listed both in its combined and separated forms.  The fifth column is an ascii encoding of the Yiddish script, available at [yiddishycode](https://github.com/skulick/yiddishycode).  It is included for convenience (mine!) for grepping the  Yiddish text using ascii characters. The six column, all empty in this case, is for the occasional places where a gloss is included for a word in PPCHY, such as `(H sof^end)`. In such cases the revised psd tree has `(H sof)` and the gloss appears in the row for `sof` in the corresponding pos listing.

3. misc: This has three files created with information about the words and characters:
   - chars.txt: the Unicode code-points and names and corresponding counts for each character in the Yiddish script conversions.  Note that all the conversions are in NFC form, so there are no combined characters.
   - words.txt: a listing of the words, in their (possibly) combined forms.
   - pos.txt: a listing of the pos tags, with the words for each tag.

However, note that these files reflect only the words in the files 1910e-grine-felder and 1947e-royte-pomerantsen (see Caveats/Warning below).

4. json: As mentioned, these include both the tree and lexical information. The above files are in fact created using these json files are input.  Each tree in a file is represented as a dict, with three elements:
   - `tree_id`: the treeid, or `notreeid` if there is no tree id (i.e., just a `CODE` tree)
   - `tree`: a flattened form of the PSD tree.
   - `leaves`: a listing of the words. (this would be better named `words` instead of `leaves`.)  Each word is itself a dict with
      - `pos`: pos tag
      - `rom`: romanized form
      - `yid`: Yiddish script form
      - `ycode`: the 1-1 ascii form of the `yid` form, as mentioned above.
      - `ltype`: either `s`, `t`, or `st`.  The first means that it's only a source - i.e., combined word (e.g., `nistho` in the above).  `t` means that it was split from a `s` word (e.g., `nisht` and `o` in the above).  `st` means that it is both source and tree - ie, it is not a word that was split.  
      - `split_before`: optionally present, and if so, it's True and means that it was split from a word before.
      - `split_after`: optionally present, and if so, it's True and means that it was spit from a word after.
      - `start`: the word index in the tree
      - `end`: only for `s` words, and together `start` and `end` are the spans indices of its component words in the tree.

# Caveats/Warning

Since the main focus is NLP applications, I was concerned, for now, only with two files - 1910e-grine-felder and 1947e-royte-pomerantsen. These make up a large part of the treebank and are being used for the training and evaluation.  Since they are more modern Yiddish, the conversion to Yiddish script is reasonable for those files.  While all the files were run through the same conversion, it is unlikely to be very meaningful for the earlier files.

# Todos

1. There are still many words, of Hebrew origin, in the two files, particularly 1947e-royte-pomerantsen, are not being properly converted to Yiddish script.  This is due to a difference in the romanized form for a Hebrew word in PPCHY and that expected in the Yiddish package.  These are in a continual state of being corrected.

# Contact

For anything related to these particular issues of conversion: `skulick@ldc.upenn.edu`



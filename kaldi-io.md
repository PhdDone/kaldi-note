Kaldi IO Notes
=============

* Similarly to perl, a filename can be replaced with - (for standard input/output) or a string such as "|gzip -c >foo.gz" or "gunzip -c foo.gz|"

* to avoid seeing the log messages you could redirect stderr to /dev/null by adding 2>/dev/null to the command line.

 ```
 echo '[ 0 1 ]' | copy-matrix --binary=false - - \
 	2>/ dev/null
 ```
 
* Table I/O: Kaldi has special I/O mechanisms for dealing with collections of objects indexed by strings. Examples of this are feature matrices indexed by utterance-ids, or speaker-adaptation transformation matrices indexed by speaker-ids. 
 * Programs that read from Tables expect a string we call an "rspecifier" (read specifier).
 * Programs that read from Tables expect a string we call an "rspecifier" that says how to read the indexed data.
 * and programs that write to Tables expect a string we call a "wspecifier" to write it.
 * It is possible to write to both an archive and script at the same time, e.g. ark,scp:foo.ark,foo.scp. The script file will be written to, with lines like "utt1 foo.ark:1016" (i.e. it points to byte offsets in the archive). This is useful when data is to be accessed in random order or in parts, but you don't want to produce lots of small files. (Our dev set is in this format
 * `scp` here should mean `script`

* Example: take a scp format rspecifier and scp format wspecifier, write matrix to stdout. 

 ```
  echo '[ 0 1 ]' | \ 
  copy-matrix 'scp:echo foo -|' 'scp,t:echo foo -|'
 ```
 * copy-matrix first paramiter is input, second is output.
 * if the specifier in on input position, then it's rspecifier.
 * `echo foo -|` will pipe `foo -` as input, `-` means stdio. Must have pipe `|` at the end.
 * Common types of rspecifiers include "ark:-", meaning read the data as an archive from the standard input, or "scp:foo.scp", meaning the script file foo.scp says where to read the data from.
 * The script-file format is, on each line, "<key> <rspecifier|wspecifier>", e.g. "utt1 /foo/bar/utt1.mat". It is OK for the rspecifier or wspecifier to contain spaces, e.g. `utt1 gunzip -c /foo/bar/utt1.mat.gz |`. or `fid cat test.input |`. Note that there must be a pipe at the end.

* another example, wspecifier `ark,t:-` means archive in text format and print to stdout

 ```
 echo '[ 0 1 ]' | copy-matrix \
 'scp,t:echo foo -|' ark,t:-
 ```
 
* rspecifier `ark:gunzip -c input.gz|` means read from an unzipped archived file.
* wspecifier `ark:|gzip -c output.gz` means write in achived format then pipe to gzip to zip it.
	
**This work is licensed under a [Creative Commons Attribution-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US). This means that you are able to copy, share and modify the work, as long as the result is distributed under the same license.**

# Concordia Python workshop - module 3 - Intorduction to Biopython   

by Mathieu Bourgey, _Ph.D_  

This Workshop is an adaptation of some interesting point of the [general Biopython tutorial](http://biopython.org/DIST/docs/tutorial/Tutorial.html)

## Learning objectives
During this wiorkshop you will learn:  

 - Manipulate Sequence Objects
 - Annotate Sequence Objects
 - Read/write Fasta file
 - Blasting Sequences
 - Multiple Alignment Sequences 



## The Seq Object
Biological sequences are arguably the central object in Bioinformatics. Biopython sequences are essentially strings of letters like **AGTACACTGGT** as seen in common biological file formats.

The Biopython `Seq` object is defined in the `Bio.Seq` module

The `Seq` object is different from traditional python strings:  

 1. It supports most of the string methods but it also comes with its specifc set of methods
  * translate() _- Turns a nucleotide sequence into a protein sequence._
  * reverse_complement() _- Returns the reverse complement sequence._
  * complement() _- Returns the complement sequence._
  * transcribe() _-Returns the RNA sequence from a DNA sequence._
  * back_transcribe() _- Returns the DNA sequence from an RNA sequence._
  * ungap() _- Return a copy of the sequence without the gap character(s)._
 2. It has an important attribute, the alphabet, which is an object describing the type of the sequence and how the characters should be interpreted. Biopython alphabet are define in the `Bio.Alphabet` module

 
[The detail API of the `Seq` object](http://biopython.org/DIST/docs/api/Bio.Seq.Seq-class.html)


[The detail API of the `Alphabet` object](http://biopython.org/DIST/docs/api/Bio.Alphabet-module.html)
 
### Sequence Manipulation
We’ll use the [IUPAC](http://www.chem.qmw.ac.uk/iupac/) alphabets here to deal with some of our favorite objects: DNA, RNA and Proteins.

We can create an ambiguous sequence with the default generic alphabet like this:

```{.python}
from Bio.Seq import Seq
my_seq = Seq("AGTACACTGGT")
my_seq
```

> Seq('AGTACACTGGT', Alphabet())

```{.python}
my_seq.alphabet
```

> Alphabet()

In the above example, we haven't specified an alphabet so we end up with
a default generic alphabet. Biopython doesn't know if this is a
nucleotide sequence or a protein rich in alanines, glycines, cysteines
and threonines. If *you* know, you should supply this information

```{.python}
from Bio.Alphabet import IUPAC
my_seq = Seq("AGTACACTGGT", IUPAC.unambiguous_dna)
my_seq
```

> Seq('AGTACACTGGT', IUPACUnambiguousDNA())

```{.python}
my_seq.alphabet
```

> IUPACUnambiguousDNA()

####  Seq as python strings

In many ways, we can deal with Seq objects as if they were normal Python strings, for example getting the length, or iterating over the elements

```{.python}
for index, letter in enumerate(my_seq):
   print("%i %s" % (index, letter))

```

> 0 A   
> 1 G   
> 2 T   
> 3 A   
> 4 C   
> 5 A   
> 6 C   
> 7 T   
> 8 G   
> 9 G   
> 10 T   

```{.python}
len(my_seq)
```

> 11

The Seq object has a `.count()` method, just like a string. Note that this means that like a Python string, this gives a non-overlapping count

```{.python}
my_seq.count("A")
```

> 3

```{.python}
my_seq.count("GT")
```

> 2

Note that this means that like a Python string, this gives a non-overlapping count:

```{.python}
Seq("AAAA").count("AA")
```

> 2

####  Slicing a Seq object

let’s get a slice of the sequence

```{.python}
my_seq[2:8]
```

> Seq('TACACT', IUPACUnambiguousDNA())

Note that it follows the normal conventions for Python strings. 
 - So the first element of the sequence is 0 (which is normal for computer science, but not so normal for biology).
 - The first item is included i.e. 2 in this case (3rd aa)
 - The last is excluded i.e 8 in this case (9th aa)

###### Exercice

**In one command could you extract the third codon positions of this DNA sequence ?**

[solution](solutions/_codonExtract.md)

#### Concatenate sequence
You can in principle add any two Seq objects together just like you can with Python strings. But `Seq` object are made for biological data so you the concatenation method only accept to merge sequences with compatible alphabets. You are allowed to concatenate a protein sequence and a DNA sequence.

```{.python}
p_seq = Seq("EVRNAK", IUPAC.protein)
d_seq = Seq('TACACT', IUPAC.unambiguous_dna)
d_seq + my_seq
```

> Seq('TACACTAGTACACTGGT', IUPACUnambiguousDNA())


```{.python}
p_seq + my_seq
```

> Traceback (most recent call last):   
> ...   
> TypeError: Incompatible alphabets IUPACProtein() and IUPACUnambiguousDNA()

**If you __realy__ want to do that, how should you do ?** [solution](solutions/_concate.md)

#### Seq as Biological strings

The `Seq` object is more than a python string with a specific alphabet, it also offers methods specific to facilitate the biology oriented analysis.

#### Complement and reverse complement
DNA is double stranded but in most case sequence are represented as single stranded molecules. for many purpose , i.e alignment, we need to compare a query sequence to reference sequence. In this case, we need to know if the reference sequence contains the query sequence in one or the other strands. 

You can easily obtain the complement or reverse complement of a Seq object using its built-in methods:

```{.python}
my_seq.complement()

```

> Seq('TCATGTGACCA', IUPACUnambiguousDNA())

```{.python}
my_seq.reverse_complement()

```

> Seq('ACCAGTGTACT', IUPACUnambiguousDNA())


There is no specific methods in Biopython to only reverse your sequence


**Do you know why and how to proceed ?**   
[solution](solutions/_reverse.md)


Note that these methods only work for dna alphabet. Trying to (reverse)complement a protein sequence will raise you an error:

```{.python}
p_seq = Seq("EVRNAK", IUPAC.protein)
p_seq.reverse_complement()

 ```
 
> Traceback (most recent call last):__ 
> ...   
> ValueError: Proteins do not have complements!

#### Transcirption, reverse transcription and translation 
First we need to clarify the transcription strand issue !   

![Transcription strand](img/Transcription_strand.png)

Biologically the transcription do a reverse complement of the template strand while inserting Uracile instead of Thymine (TCAG → CUGA) to give the RNA.

However, in Biopython and bioinformatics in general, we typically work directly with the coding strand because this means we can get the mRNA sequence just by switching T → U

Let's do a simple transcription of our sequence:

```{.python}
r_seq=my_seq.transcribe()
r_seq

```

> Seq('AGUACACUGGU', IUPACUnambiguousRNA())

And a reverse transcription of the resulting sequence:


```{.python}
r_seq.back_transcribe()

```
> Seq('AGTACACTGGT', IUPACUnambiguousDNA())

As you can see, all this does is switch T -> U or U -> T and adjust the alphabet.


###### Exercice

**Could you generate the mRNA from this template strand sequence: __3'-TACCGGTAACATTACCCGGCGACTTTCCCACGGGCTATC-5'__ ?**  

[solution](solutions/_mRNA.md)


Now let’s translate this mRNA into the corresponding protein sequence 

```{.python}
p_seq = r_seq.translate()
p_seq
```

> Seq('MAIVMGR*KGAR*', HasStopCodon(IUPACProtein(), '*'))

Note that Biopython also allow to directly translate from DNA sequence. In this case use the coding strand DNA sequence.

You should also notice in the above protein sequences that in addition to the end stop character, there is an internal stop as well. This is due to the use of the wrong translation table in this case.

The translation tables available in Biopython are based on those from the [NCBI](http://www.ncbi.nlm.nih.gov/Taxonomy/Utils/wprintgc.cgi). By default, translation will use the standard genetic code (NCBI table id 1). 

In our case, we are dealing with a mitochondrial sequence. We need to tell the translation function to use the relevant genetic code instead:

```{.python}
p_seq = r_seq.translate(table="Vertebrate Mitochondrial")
p_seq
```

> Seq('MAIVMGRWKGAR*', HasStopCodon(IUPACProtein(), '*'))

Also you may want to translate the nucleotides up to the first in frame stop codon, and then stop. In this case you should use to_stop method:


```{.python}
r_seq.translate(to_stop=True)
```

> Seq('MAIVMGR', IUPACProtein())

```{.python}
r_seq.translate(table=2, to_stop=True)
```

> Seq('MAIVMGRWKGAR', IUPACProtein())

Notice that when you use the to_stop argument, the stop codon itself is not translated, Notice also that you can specify the table using the NCBI table number which is shorter.

## The SeqRecord object
Immediately “above” the `Seq` class is the Sequence Record or `SeqRecord` class, defined in the `Bio.SeqRecord` module. This class allows higher level features to be associated with the sequence.

[The detail API of the `SeqRecord` object](http://biopython.org/DIST/docs/api/Bio.SeqRecord.SeqRecord-class.html)

A SeqRecord object holds a sequence and information about it.

**Main attributes:**

        * .id - Identifier such as a locus tag or an accesion number (string)
        * .seq - The sequence itself (Seq object or similar)

**Additional attributes:**

        * .name - Sequence name, e.g. gene name (string)
        * .description - A human readable description or expressive name for the sequence (string)
        * .letter_annotations - Per letter/symbol annotation (restricted dictionary). This holds Python sequences (lists, strings or tuples) whose length matches that of the sequence. A typical use would be to hold a list of integers representing sequencing quality scores, or a string representing the secondary structure.
        * .features - Any (sub)features defined (list of `SeqFeature` objects), i.e location, type or strand...
        * .annotations - Further information about the whole sequence (dictionary). Most entries are strings, or lists of strings. This allows the addition of more “unstructured” information to the sequence.
        * .dbxrefs - List of database cross references (list of strings)

Using a `SeqRecord` object is not very complicated, since all of the information is presented as attributes of the class. Usually you won’t create a `SeqRecord` “by hand”, but instead you a specific Class to read in a sequence file for you (presented in the next section). However, creating `SeqRecord` can be quite simple.


### Manually creating a seqRecord

To create a `SeqRecord` at a minimum you just need a `Seq` object:

```{.python}
from Bio.SeqRecord import SeqRecord
simple_seq_r = SeqRecord(my_seq)
simple_seq_r

```
> SeqRecord(seq=Seq('AGTACACTGGT', IUPACUnambiguousDNA()), id='<unknown id>', name='<unknown name>', description='<unknown description>', dbxrefs=[])

We can also manually pass the id, name and description to the object

```{.python}
simple_seq_r.id = "THX1138"
simple_seq_r.name = "THX 1138 4EB"
simple_seq_r.description = "Made up sequence I wish I could write a paper about"
simple_seq_r

```

> SeqRecord(seq=Seq('AGTACACTGGT', IUPACUnambiguousDNA()), id='THX1138', name='THX 1138 4EB', description='Made up sequence I wish I could write a paper about', dbxrefs=[])

Including an identifier is very important if you want to output your SeqRecord to a file.

The `SeqRecord` has an dictionary attribute annotations. This is used for any miscellaneous annotations that doesn’t fit under one of the other more specific attributes. Adding annotations is easy, and just involves dealing directly with the annotation dictionary:

```{.python}
simple_seq_r.annotations["evidence"] = "None. I just made it up."
simple_seq_r.annotations

```
> {'evidence': 'None. I just made it up.'}

Working with per-letter-annotations is similar, letter_annotations is a dictionary like attribute which will let you assign any Python sequence (i.e. a string, list or tuple) which has the same length as the sequence.

```{.python}
import random
simple_seq_r.letter_annotations["phred_quality"] =  random.sample(xrange(1, 50),len(simple_seq_r))
simple_seq_r.letter_annotations

```

> {'phred_quality': [22, 23, 3, 2, 29, 11, 34, 44, 5, 33, 16]}

The `format()` method of the `SeqRecord` class gives a string containing your record formatted using one of the output file formats supported. 

```{.python}
simple_seq_r.format('fasta')

```
> '>THX1138 Made up sequence I wish I could write a paper about\nAGTACACTGGT\n'

```{.python}
simple_seq_r.format('fastq')
```

> '@THX1138 Made up sequence I wish I could write a paper about\nAGTACACTGGT\n+\n78$#>,CM&B1\n'

## The SeqIO Class
The `SeqIO` Class provide a simple interface for working with assorted sequence file formats in a uniform way.

The “catch” is that you have to work with `SeqRecord` objects, which contain a `Seq` object with format specific annotation like an identifier and description.


[The detail API of the `SeqIO` module](http://biopython.org/DIST/docs/api/Bio.SeqIO-module.html)

### Reading or parsing sequence files 

The main method of the `SeqIO` module is __.parse()__  is used to read in sequence data as `SeqRecord` objects. This function expects two arguments:

    1. A handle to read the data from, or a filename.
    2. A lower case string specifying sequence format. See http://biopython.org/wiki/SeqIO for a full listing of supported formats. 
    
There is an optional argument alphabet to specify the alphabet to be used. This is useful for file formats like FASTA where otherwise Bio.SeqIO will default to a generic alphabet.

The __.parse()__ methods is typically used with a for loop like this:

```{.python}
from Bio import SeqIO
for seq_record in SeqIO.parse("data/NC_000913.fna","fasta"):
    print(seq_record.id)
    print(repr(seq_record.seq))
    print(len(seq_record))
    print(len(seq_record.features))

```

> gi|556503834|ref|NC_000913.3|   
> Seq('AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAG...TTC', SingleLetterAlphabet())   
> 4641652   
> 0


If instead you wanted to load a GenBank format:

```{.python}
for seq_record in SeqIO.parse("data/NC_000913.gbk","genbank"):
    print(seq_record.id)
    print(repr(seq_record.seq))
    print(len(seq_record))
    print(len(seq_record.features))

```

> gi|556503834|ref|NC_000913.3|   
> Seq('AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAG...TTC', SingleLetterAlphabet())   
> 4641652   
> 22996 


Notice that in the 2 examples the `SeqRecord` object contain the same central information and that mostly the difference is in the amount of information provided in the annotations (__.features__) 

Similarly, if you wanted to read in a file in another file format, you would just need to change the format string as appropriate. For example “swiss” for SwissProt files or “embl” for EMBL text files. There is a full listing on the [Biopython wiki page](http://biopython.org/wiki/SeqIO)

Another very common way to use a Python iterator is within a list comprehension

```{.python}
identifiers = [seq_record.id for seq_record in SeqIO.parse("data/patato_pep.fasta","fasta")]
identifiers

```

> ['PGSC0003DMP400067339', 'PGSC0003DMP400027454', 'PGSC0003DMP400020381', 'PGSC0003DMP400022612', 'PGSC0003DMP400040011', 'PGSC0003DMP400032361', 'PGSC0003DMP400030628', 'PGSC0003DMP400028584', 'PGSC0003DMP400060824', 'PGSC0003DMP400037883']

We usually use a for loop to iterate over all the records one by one. The object returned by `SeqIO` is actually an iterator which returns `SeqRecord` objects. You get to access each record in turn, but once and only once. The plus point is that an iterator can save you memory when dealing with large files.

Sometime you need to be able to access the records in any order. In that case you could acces the records using a list or dictionary process. But be carefull because you could easily carsh or slowdown python when using this methods on large data sets.

#### The list approach
We can turn the record iterator into a list of SeqRecord objects using the built-in Python function __list()__ 


```{.python}
records = list(SeqIO.parse("data/patato_pep.fasta","fasta"))
records[3]
```

> SeqRecord(seq=Seq('MLPFESIEEASMSLGRNLTFGETLWFNYTADKSDFYLYCHNTIFLIIFYSLVPL...SE*', SingleLetterAlphabet()), id='PGSC0003DMP400022612', name='PGSC0003DMP400022612', description='PGSC0003DMP400022612 PGSC0003DMT400033236 Protein', dbxrefs=[])

#### The dictionary approach
`SeqIO` module has three related functions which allow dictionary like random access to a multi-sequence file. There is a trade off here between flexibility and memory usage. In summary:  

   - `.to_dict()` is the most flexible but also the most memory demanding option. This is basically a helper function to build a normal Python dictionary with each entry held as a SeqRecord object in memory, allowing you to modify the records.
   -  `.index()` is a useful middle ground, acting like a read only dictionary and parsing sequences into SeqRecord objects on demand.
   - `.index_db()` also acts like a read only dictionary but stores the identifiers and file offsets in a file on disk (as an SQLite3 database), meaning it has very low memory requirements, but will be a little bit slower. 


Let's test the different functions

You can use the function `.to_dict()` to make a SeqRecord dictionary (in memory). By default this will use each record’s identifier (i.e. the .id attribute) as the key

```{.python}
records_dict = SeqIO.to_dict(SeqIO.parse("data/patato_pep.fasta","fasta"))
records_dict.keys()

```

> ['PGSC0003DMP400030628', 'PGSC0003DMP400032361', 'PGSC0003DMP400027454', 'PGSC0003DMP400060824', 'PGSC0003DMP400040011', 'PGSC0003DMP400037883', 'PGSC0003DMP400022612', 'PGSC0003DMP400020381', 'PGSC0003DMP400067339', 'PGSC0003DMP400028584']

```{.python}
records_dict['PGSC0003DMP400020381']
```

> SeqRecord(seq=Seq('MLEKDSRDDRLDCVFPSKHDKDSVEEVSSLSSENTRTSNDCSRSNNVDSISSEV...KY*', SingleLetterAlphabet()), id='PGSC0003DMP400020381', name='PGSC0003DMP400020381', description='PGSC0003DMP400020381 PGSC0003DMT400029984 Protein', dbxrefs=[])

`.to_dict()` is very flexible because it holds everything in memory. The size of file you can work with is limited by your computer’s RAM. In general, this will only work on small to medium files.

---------------

For larger files you should consider `.index()`, which works a little differently. Although it still returns a dictionary like object, this does not keep everything in memory. Instead, it just records where each record is within the file and when you ask for a particular record, it then parses it on demand.


```{.python}
records_dict = SeqIO.index("data/patato_pep.fasta","fasta")
list(records_dict.keys())

```

> ['PGSC0003DMP400030628', 'PGSC0003DMP400032361', 'PGSC0003DMP400027454', 'PGSC0003DMP400060824', 'PGSC0003DMP400040011', 'PGSC0003DMP400037883', 'PGSC0003DMP400022612', 'PGSC0003DMP400020381', 'PGSC0003DMP400067339', 'PGSC0003DMP400028584']

Note that in this case the `.keys()` function return an iterator and we need to use the list function to get the key values

```{.python}
records_dict['PGSC0003DMP400020381']
```

> SeqRecord(seq=Seq('MLEKDSRDDRLDCVFPSKHDKDSVEEVSSLSSENTRTSNDCSRSNNVDSISSEV...KY*', SingleLetterAlphabet()), id='PGSC0003DMP400020381', name='PGSC0003DMP400020381', description='PGSC0003DMP400020381 PGSC0003DMT400029984 Protein', dbxrefs=[])

Note that `.index()` won’t take a handle, but only a filename. 

-----------------

`.index_db()` work on even extremely large files since it stores the record information as a file on disk (using an SQLite3 database) rather than in memory. Also, we can index multiple files together (providing all the record identifiers are unique).

`.index_db()` function takes three required arguments:

    - Index filename
    - List of sequence filenames to index (or a single filename)
    - File format (lower case string as used in the rest of the SeqIO module). 
    
```{.python}
patato_pep = SeqIO.index_db("patato_pep.idx", "data/patato_pep.fasta","fasta")
patato_pep.keys()
```

> ['PGSC0003DMP400020381', 'PGSC0003DMP400022612', 'PGSC0003DMP400027454', 'PGSC0003DMP400028584', 'PGSC0003DMP400030628', 'PGSC0003DMP400032361', 'PGSC0003DMP400037883', 'PGSC0003DMP400040011', 'PGSC0003DMP400060824', 'PGSC0003DMP400067339']

```{.python}
patato_pep['PGSC0003DMP400040011']
```

> SeqRecord(seq=Seq('MECDTEDSEDNSNIQADSNHRLVKFVIPGNNLLDQTKSSSTKVVLIFLESVEIL...NF*', SingleLetterAlphabet()), id='PGSC0003DMP400040011', name='PGSC0003DMP400040011', description='PGSC0003DMP400040011 PGSC0003DMT400059441 Protein', dbxrefs=[])


**So, which of these methods should you use and why ?** [solution](solutions/_seqIO1.md) 

### Writing sequence files

The `.write()` is used to output sequences (writing files). This function taking three arguments: 

 - Some SeqRecord objects. 
 - A handle or filename to write to. 
 - A sequence format.

First, let's write a sequence into the file __testOut.fa__ :

```{.python}
import os
SeqIO.write(simple_seq_r, "testOut.fa",  "fasta")
os.system("cat testOut.fa")

```
> \>THX1138 Made up sequence I wish I could write a paper about   
> AGTACACTGGT

then, let's write a set of patato sequences into the file __testOut.fa__ :

```{.python}
for seq_record in SeqIO.parse("data/patato_pep.fasta","fasta") : 
  SeqIO.write(seq_record, "testOut.fa",  "fasta")

os.system("cat testOut.fa")

```
> \>PGSC0003DMP400037883 PGSC0003DMT400056292 Protein   
> MMIGRDPEIWENPEEFIPERFLNSDIDYFKGQNFELIPFGAGRRGCPGIALGVATINLIL   
> SNLLYAFDWELPHGMIKEDIDTDGLPGLAMHKKNALCLVPKNYTHT*


**Do you notice something strange in testOut.fa ? and explain why ?** [solution](solutions/_seqIO2.md)


###### Exercice

**Could you write the content data/patato_pep.fasta into testOut.fa ?** [solution](solutions/_seqIO3.md)


## The Blast Class
Everybody loves BLAST ! How can it get any easier to do comparisons between one of your sequences and every other sequence in the known world?

If you don't know Blast please explore http://blast.ncbi.nlm.nih.gov/Blast.cgi

Dealing with BLAST can be split up into two steps, both of which can be done from within Biopython:

 1. Running BLAST for your query sequence(s), and getting some output. 
 2. Parsing the BLAST output in Python for further analysis.

To do that Biopython provide the specific `Blast` module.

[The detail API of the `Blast` module](http://biopython.org/DIST/docs/api/Bio.Blast-module.html)
 
There are lots of ways you can run BLAST, especially you can run BLAST locally (on your own machine), or run BLAST remotely (on another machine, typically the NCBI servers).

In this workshop we will only focus on the remote way to run blast.


### Running BLAST on NCBI servers
The specific sub-module `Blast.NCBIWWW` allows to call the online version of BLAST through is main function `qblast()`.

This function has three mandatory arguments:

 1. The blast program to use for the search, as a lower case string. Currently qblast only works with blastn, blastp, blastx, tblast and tblastx.
 2. The databases to search against, as a lower case string.
 3. The query sequence as a string. This can either be the sequence itself, the sequence in fasta format, or an identifier like a GI number.
 
Note that the default settings on the NCBI BLAST website are not quite the same as the defaults on QBLAST. If you get different results, you’ll need to check the parameters (e.g., the expectation value threshold and the gap values).
 
```{.python}
from Bio.Blast import NCBIWWW
result_handle = NCBIWWW.qblast("blastp", "nr", patato_pep['PGSC0003DMP400040011'].seq)
```

Supplying just the sequence means that BLAST will assign an identifier for your sequence automatically. You might prefer to use the SeqRecord object’s format method to make a FASTA string (which will include the existing identifier):

```{.python}
result_handle = NCBIWWW.qblast("blastp", "nr", patato_pep['PGSC0003DMP400040011'].format("fasta"))
```

### Writing and reading
Whatever arguments you give the qblast() function, you should get back your results in a handle object (by default in XML format). The next step would be to parse the XML output into Python objects representing the search results, but you might want to save a local copy of the output file first. 

To output the blast result we use the `read()` function:

```{.python}
save_file = open("my_blast.xml", "w")
save_file.write(result_handle.read())
save_file.close()
os.system("head my_blast.xml")
```

> \<?xml version="1.0"?\>   
> \<!DOCTYPE BlastOutput PUBLIC "-//NCBI//NCBI BlastOutput/EN" "http://www.ncbi.nlm.nih.gov/dtd/NCBI_BlastOutput.dtd">   
> \<BlastOutput\>   
> &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_program\>blastp\</BlastOutput_program\>   
> &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_version\>BLASTP 2.3.1+\</BlastOutput_version\>   
> &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_reference\>Stephen F. Altschul, Thomas L. Madden, Alejandro A. Sch&amp;auml;ffer, Jinghui Zhang, Zheng Zhang, Webb Miller, and David J. Lipman (1997), &quot;Gapped BLAST and PSI-BLAST: a new generation of protein database search programs&quot;, Nucleic Acids Res. 25:3389-3402.\</BlastOutput_reference\>   
>  &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_db\>nr\</BlastOutput_db\>   
>  &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_query-ID\>Query_56965\</BlastOutput_query-ID\>   
>  &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_query-def\>**PGSC0003DMP400040011 PGSC0003DMT400059441 Protein**\</BlastOutput_query-def\>   
>  &nbsp;&nbsp;&nbsp;&nbsp;\<BlastOutput_query-len\>222\</BlastOutput_query-len\>   


We need to be a bit careful since we can use result_handle.read() to read the BLAST output only once – calling result_handle.read() again returns an either an empty string .


```{.python}
result_handle.read()
```

> ''

After doing this, the results are in the file my_blast.xml and the original handle has had all its data extracted, so we can close it:


```{.python}
result_handle.close()
result_handle.read()
```

> Traceback (most recent call last):   
> ...   
> ValueError: I/O operation on closed file


To acces our blast results we can read the save data using the `parse` function of the BLAST parser. `parse` takes a file-handle-like object as argument so we just need to traditionally open the blast result file:   


```{.python}
result_handle = open("my_blast.xml")
```

### Parsing BLAST output
BLAST can generate output in various formats, such as XML, HTML, and plain text. Unfortunately, the BLAST output in HTML or plain text kept changing, each time breaking the original Biopython parsers. the HTML BLAST parser has been removed, but the plain text BLAST parser is still available but deprecated.

Biopython recommend to parse the output in XML format, which can be generated by recent versions of BLAST. Not only is the XML output more stable than the plain text and HTML output, it is also much easier to parse automatically, making Biopython a whole lot more stable.

We can get BLAST output in XML format in various ways (internet loccaly, etc...). For the parser, it doesn’t matter how the output was generated, as long as it is in the XML format.

The specific sub-module `Blast.NCBIXML` allows to parse BLAST XMLs.

```{.python}
from Bio.Blast import NCBIXML
```

Just like the `SeqIO`object, we have a pair of input functions, 

 - `.read()` for when you have exactly one object
 - `.parse()`to generate an iterator for when you can have lots of objects 


But instead of getting SeqRecord objects, we get BLAST record objects.

We’ve got a handle, we are ready to parse the output. The code to parse it is really quite small if we expect a single BLAST result or if ywe want to parse only the first (next) results:

```{.python}
blast_record = NCBIXML.read(result_handle)
```

or, if you have lots of results you want to 

```{.python}
result_handle.seek(0)
blast_records = NCBIXML.parse(result_handle)
```

** In the example above why are we using the `.seek(0)` methods ?** [solution](solutions/_blast1.md)

Note though that you can step through the BLAST records only once. Usually, from each BLAST record you would save the information that you are interested in. If you want to save all returned BLAST records, you can convert the iterator into a list:

```{.python}
blast_record_list = list(blast_records)
```

Now you can access each BLAST record in the list with an index as usual.

** What should we be careful of when storing the blast_records into a list ? ** [solution](solutions/_blast2.md)

I guess by now you’re wondering what is in a BLAST record !

### The BLAST record class
A BLAST Record contains everything you might ever want to extract from the BLAST output. Right now we’ll just show an example of how to get some info out of the BLAST report, but if you want something in particular that is not described here, look at the info on the [BLAST record class](http://biopython.org/DIST/docs/api/Bio.Blast.Record-module.html) in detail.

A BLAST record contains a variety of informations in sub-Classes (partial list):  

- Header - Saves information from a blast header.
- Description - Stores information about one hit in the descriptions section.
- Alignment - Stores information about one hit in the alignments section.
- HSP - Stores information about one hsp in an alignment hit.
- Parameters - Holds information about the parameters.
- Blast - Saves the results from a blast search.


Now let’s just print out some summary info about the hits in our blast report greater than a particular threshold (1e-35).

```{.python}
E_VALUE_THRESH = 1e-35
for alignment in blast_record.alignments:
     for hsp in alignment.hsps:
         if hsp.expect < E_VALUE_THRESH:
             print '****Alignment****'
             print 'sequence:' + alignment.title
             print 'length:', alignment.length
             print 'e value:', hsp.expect
             print hsp.query[0:75] + '...'
             print hsp.match[0:75] + '...'
             print hsp.sbjct[0:75] + '...'
            
```

> \*\*\*\*Alignment\*\*\*\*   
> sequence:gi|971553515|ref|XP_015164971.1| PREDICTED: uncharacterized protein LOC102590883 isoform X2 [Solanum tuberosum]   
> length: 776   
> e value: 2.54083e-38   
> MECDTEDSEDNSNIQAD------------------SNHRLVKFVIPGNNL---LDQTK----SSSTKVVLIFLE-...   
> ME D EDSED+SNIQAD                   N +L    I        L+Q +    +S+  +    LE ...   
> MESDPEDSEDSSNIQADIQQPPSPEICNTGEQPPRPNKKLFNKGIVCRKFERHLEQNEEDVLTSAITLFAFSLEK...   
> \*\*\*\*Alignment\*\*\*\*   
> sequence:gi|565365084|ref|XP_006349247.1| PREDICTED: uncharacterized protein LOC102590883 isoform X1 [Solanum tuberosum]   
> length: 792   
> e value: 1.91084e-36   
> SVEILTSVAKKVHMQLQNAEFHIQADMGRITSLNKLKRKHVEEVLQEKLQHLSSIYERCKEEVTRHLQDCKSTLQ...   
> S EIL SVA+K+HMQLQNAEF IQADMGRITSLNK KRKHVEEVLQEK QHLS+IYER KEEVTRHLQDCKSTL+...   
> SAEILNSVAEKIHMQLQNAEFQIQADMGRITSLNKSKRKHVEEVLQEKQQHLSAIYERFKEEVTRHLQDCKSTLE...   


Basically, we can do anything we want to with the info in the BLAST report once we have parsed it. This will, of course, depend on what we want to use it for.

[Here are is UML class diagrams for the Blast record class] (http://biopython.org/DIST/docs/tutorial/Tutorial.html#fig:blastrecord)

[Here are is UML class diagrams for the PSIBlast class] (http://biopython.org/DIST/docs/tutorial/Tutorial.html#fig:psiblastrecord)

## Multiple Alignment Sequences
Multiple Sequence Alignments are a collection of multiple sequences which have been aligned together, usually with the insertion of gap characters, such that all the sequence strings are the same length. Alignment can be regarded as a matrix of letters, where each row is held as a SeqRecord object internally.

the `MultipleSeqAlignment` object holds this kind of data, and the `AlignIO` module fis used for reading and writing them as various file formats.

[The detail API of the `AlignIOt` module](http://biopython.org/DIST/docs/api/Bio.AlignIO-module.html)

In this workshop we won't cover the specific modules dedicated to command line wrappers for common multiple sequence alignment tools !

### Parsing or Reading Sequence Alignments
`AlignIO` contains 2 functions for reading in sequence alignments:

- `read()` - will return a single `MultipleSeqAlignment` object
- `parse()` - will return an iterator which gives `MultipleSeqAlignment` objects

Both functions expect two mandatory arguments:

- A string specifying a handle to an open file or a filename.
- A lower case string specifying the alignment format. 

[See here for a full listing of supported formats](http://biopython.org/wiki/AlignIO)


And 2 optional arguments:

- The seq_count argument (integer) specifying the number of alignment in case of ambiguous file formats.
- The alphabet argument allowing to specify the expected alphabet.

#### Singles alignments
Let's start with a single alignments file which contains the alignments of the 10 differents patato petitides we used previously (file __data/muscle-patato_pep.clw__). The alignments were generated using MUSCLE (MUltiple Sequence Comparison by Log-Expectation) which is a refeence for aligning amino-acide sequences and output in the CLUSTAL format.

First let's look what is inside the file

```{.python}
os.system("head data/muscle-patato_pep.clw")
```

> CLUSTAL multiple sequence alignment by MUSCLE (3.8)   
>   
>   
> PGSC0003DMP400022612      ------------------------------------------------------------   
> PGSC0003DMP400060824      ---------------MQIFVKTLTGKTITLEVESSDTIDNVKAKIQDKEGIPPDQQRL--   
> PGSC0003DMP400067339      ---------------MGVWKDSNYGKGVIIGVIDT--GILPDHPSFSDVGMPPPPAKWKG   
> PGSC0003DMP400027454      MPHPTQVVALLKAQQIRHVRLFNADRGMLLALANTGIKVAVSVPNEQILGVGQSNTTAAN   
> PGSC0003DMP400028584      ------------------MSTSVEPNGAVL--------------LDSTAGSGGGVANSNG   
> PGSC0003DMP400030628      ------------------------------------------------------------   
> PGSC0003DMP400020381      ------------------MLEKDSRDDRLDCVFPS---KHDKDSVEEVSSLSSENTRTSN   

We can load this file as follows :

```{.python}
from Bio import AlignIO
aln_patato = AlignIO.read("data/muscle-patato_pep.clw", "clustal")
print aln_patato
```

> SingleLetterAlphabet() alignment with 10 rows and 1294 columns   
> --------------------------------------------...--- PGSC0003DMP400022612   
> ---------------MQIFVKTLTGKTITLEVESSDTIDNVKAK...GGF PGSC0003DMP400060824   
> ---------------MGVWKDSNYGKGVIIGVIDT--GILPDHP...LDK PGSC0003DMP400067339   
> MPHPTQVVALLKAQQIRHVRLFNADRGMLLALANTGIKVAVSVP...TSS PGSC0003DMP400027454   
> ------------------MSTSVEPNGAVL--------------...--- PGSC0003DMP400028584   
> --------------------------------------------...--- PGSC0003DMP400030628   
> ------------------MLEKDSRDDRLDCVFPS---KHDKDS...--- PGSC0003DMP400020381   
> --------------------------------------------...--- PGSC0003DMP400040011   
> MEIGLAVGGAFLSSALNVLFDRLAPHGDLLNMFQKHTDDVQLFE...YL- PGSC0003DMP400032361   
> --------------------------------------------...--- PGSC0003DMP400037883   

Note in the above output the sequences have been truncated. We could instead write our own code to format this as we please by iterating over the rows as `SeqRecord` objects:

```{.python}
for record in aln_patato:
   print("%s - %s" % (record.seq[1:60], record.id))

```

> ----------------------------------------------------------- - PGSC0003DMP400022612   
> --------------MQIFVKTLTGKTITLEVESSDTIDNVKAKIQDKEGIPPDQQRL-- - PGSC0003DMP400060824   
> --------------MGVWKDSNYGKGVIIGVIDT--GILPDHPSFSDVGMPPPPAKWKG - PGSC0003DMP400067339   
> PHPTQVVALLKAQQIRHVRLFNADRGMLLALANTGIKVAVSVPNEQILGVGQSNTTAAN - PGSC0003DMP400027454   
> -----------------MSTSVEPNGAVL--------------LDSTAGSGGGVANSNG - PGSC0003DMP400028584   
> ----------------------------------------------------------- - PGSC0003DMP400030628   
> -----------------MLEKDSRDDRLDCVFPS---KHDKDSVEEVSSLSSENTRTSN - PGSC0003DMP400020381   
> ----------------------------------------------------------- - PGSC0003DMP400040011   
> EIGLAVGGAFLSSALNVLFDRLAPHGDLLNMFQKHTDDVQLFEKLGDILLGLQIVLSDA - PGSC0003DMP400032361   
> ----------------------------------------------------------- - PGSC0003DMP400037883   

With any supported file format, we can load an alignment in exactly the same way just by changing the format string. For example, use “phylip” for PHYLIP files, “nexus” for NEXUS files or “emboss” for the alignments output by the EMBOSS tools.


#### Multiple Alignments
In general alignments files can contain multiples alignment, and to read these files we must use the `AlignIO.parse()` function.

Suppose you have a multiple alignments file in PHYLIP format (dummy alignments) :

```{.python}
os.system("head data/dummy_aln.phy")
```

> &nbsp;&nbsp;&nbsp;&nbsp;5&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6   
> Alpha     AAACCA   
> Beta      AAACCC   
> Gamma     ACCCCA   
> Delta     CCCAAC   
> Epsilon   CCCAAA   
> &nbsp;&nbsp;&nbsp;&nbsp;5&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6   
> Alpha     AAACAA   
> Beta      AAACCC   
> Gamma     ACCCAA   

If we wanted to read this in using `AlignIO` we could use:

```{.python}
aln_dummy = AlignIO.parse("data/dummy_aln.phy", "phylip")
for alignment in aln_dummy:
    print alignment
    print ""

```

> SingleLetterAlphabet() alignment with 5 rows and 6 columns   
> AAACCA Alpha   
> AAACCC Beta   
> ACCCCA Gamma   
> CCCAAC Delta   
> CCCAAA Epsilon   
>    
> ...   
> SingleLetterAlphabet() alignment with 5 rows and 6 columns   
> AAAAAC Alpha   
> AAACCC Beta   
> AACAAC Gamma   
> CCCCCA Delta   
> CCCAAC Epsilon   
>    


The `.parse()` function returns an iterator. If we want to keep all the alignments in memory at once, then we need to turn the iterator into a list:

```{.python}
alignments = list(AlignIO.parse("data/dummy_aln.phy", "phylip"))
second_aln = alignments[1]
print second_aln

```

> SingleLetterAlphabet() alignment with 5 rows and 6 columns   
> AAACAA Alpha   
> AAACCC Beta   
> ACCCAA Gamma   
> CCCACC Delta   
> CCCAAA Epsilon   


### Writing Alignments
Now we’ll look at `AlignIO.write()` which is for alignments output (writing files). 

This function takes 3 arguments: 
- Some MultipleSeqAlignment objects 
- A string specifying a handle or a filename to write to
- A lower case string specifying the sequence format.

We start by creating a MultipleSeqAlignment object the hard way (by hand). Note we create some SeqRecord objects to construct the alignment from.

```{.python}
from Bio.Alphabet import generic_dna
from Bio.Align import MultipleSeqAlignment
align1 = MultipleSeqAlignment([
    SeqRecord(Seq("ACTGCTAGCTAG", generic_dna), id="toto"),
    SeqRecord(Seq("ACT-CTAGCTAG", generic_dna), id="titi"),
    SeqRecord(Seq("ACTGCTAGDTAG", generic_dna), id="tata"),
])

print align1

```

> DNAAlphabet() alignment with 3 rows and 12 columns   
> ACTGCTAGCTAG toto   
> ACT-CTAGCTAG titi   
> ACTGCTAGDTAG tata   

Now let's try to output, in phylip format, these alignments in a file with patato peptide alignments.


```{.python}
my_alignments = [align1, aln_patato]
AlignIO.write(my_alignments, "mixed.phy", "phylip")

```

> Traceback (most recent call last):
>  ...


**What is the source of the error ?** [solution](solutions/_align1.md)

**How can we resolve it ?** [solution](solutions/_align2.md)


Note - As for `SeqIO.wrtie()`, if you tell the `AlignIO.write()` function to write to a file that already exists, the old file will be overwritten without any warning.


###### Exercice

**Could you write an alignment converter function (reading,writing) ?** [solution](solutions/_align3.md)



# The Other interesting module of Biopython Object
During this workshop we only scratch the surface of the full potential of Biopython. Many other interesting modules are available, between others:

|module|task|
|---|---|
|Bio.Aff|Deal with Affymetrix related data such as cel files|
|Bio.Emboss|Code to interact with the EMBOSS programs|
|Bio.Entrez|Provides code to access NCBI over the WWW|
|Bio.HMM|A selection of Hidden Markov Model code|
|Bio.KDTree|KD tree data structure for searching N-dimensional vectors|
|Bio.KEGG|This module provides code to work with data from the KEGG database|
|Bio.PDB|Classes that deal with macromolecular crystal structures|
|Bio.Pathway|BioPython Pathway module|
|Bio.Phylo|Package for working with phylogenetic trees|
|Bio.PopGen|PopGen: Population Genetics and Genomics library in Python|
|Bio.SeqUtils|Miscellaneous functions for dealing with sequences|
|...||

**I strongly encourage you to look at he full list of module and to explore those you found relevent with your own research.**

# Homework
 
 You will find a file called [sequence.gbk](data/sequence.gbk) which contain the sequence of Mystery cRNA. The aim of the excerice is to create a script that will take in input any gbk file of a cDNA and do the following:  
- Extract the DNA sequence
- Translate it into a RNA sequence
- Transcript it into a protein sequence
- Blast the protein sequence against the nr database
- Retreive the name of the gene (if possible).
- output every sequence in a specific file using the name of the gene (or unknown if not possible)

You should try to comment your code
You should try to separated each step in a specific function or method

**There are not one unique solution to do that ! Be creative ! I will add my own implentation soon**

# Aknowledgments
This tutorial is an adaptation of some part of the one Biopython Tutorial avaialable on-line. I would like to thank the Biopytho team for their fabulous work.

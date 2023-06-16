# Guide for nextflow concepts

## Structure of a process

```groovy
params.data = "World"

process EXAMPLE {

  script:
  """
  echo 'Hello world!\nHola mundo!\nCiao mondo!\nHallo Welt!' > file
  cat file | head -n 1 | head -c 5 > chunk_1.txt
  gzip -c chunk_1.txt  > chunk_archive.gz
  """
}

process PYSTUFF {

    // specify output to standard output
    output:
    stdout

    // by default script is in bash, but can use other languages
    script:
    """
    #!/usr/bin/env python

    x = 'Hello'
    y = 'world!'
    print ("%s - %s" % (x,y))
    """
}

process FOO {
    
    script:
    """
    echo Hello params.data
    """
}

process FOO2 {

    script:
    // escape an environment variable when using double quotes
    """
    echo "The current working directory is \$PWD"
    """
}

process BAR {
    script:
    """
    echo $PATH | tr : '\\n'
    """
}

workflow {
    PYSTUFF() | view {"$it"} // <- output can be seen with view operator
    EXAMPLE()
    FOO()
    FOO2()
    BAR()
}
```

### "hello world" example

```groovy
#!usr/bin/env nextflow

params.greeting = 'Hello world! My name is '
params.name = "Miguel"
greetin_ch = Channel.of(params.greeting + params.name)
    // <- The '+' operator concatenates strings without space

process SPLITLETTERS {
    input:
    val x

    output:
    path 'chunk_*'

    """
    printf '$x' | split -b 6 - chunk_
    """
}

process CONVERTTOUPPER {
    input:
    path y

    output:
    stdout
    
    """
    cat $y | tr '[a-z]' '[A-Z]'
    """
}

process REVERSEORDER {
    debug true 

    input:
    path x

    output:
    stdout

    script:
    """
    rev $x
    """
}

workflow {
    letters_ch1 = SPLITLETTERS(greetin_ch)
    results_ch1 = CONVERTTOUPPER(letters_ch1.flatten())
    results_ch1.view()
}
```

## Channel factories

```groovy
// channel.of() factory produces a queue channel

channel
    // 1..23 means all digits starting at 1 and ending at 23
    .of(1..23, 'X', 'Y')
    .view()

// channel.fromList() emits the elements of a list as arguments

list = ['Hello', "World"]
channel
    .fromList(list)
    .view()

// channel.fromPath() emits file(s) matching a glob pattern 

channel
    .fromPath('../data/meta/*.csv')
    .view()

channel
    .fromPath('../data/ggal/**.fq', hidden:true)
    .view()

/*  channel.fromFilePairs() emits paris of files matching a pattern. Channel contains  
    tuples, with a first element ID key and a matching pair of values. */

channel
    .fromFilePairs('../data/ggal/*_{1,2}.fq', hidden:true)
    // must contain at least a star(*) wildcard
    .view()

channel
    .fromFilePairs('../data/ggal/*_{1,2}.fq', flat:true)
    // flat option removes the grouping ID key
    .view()

/*
    channel.fromSRA() looks for matching fastq files in the NCBI SRA repository. 
    Requires an API key to obtain data. File pairs ara managed the same way as if using
    the .fromFilePairs() method. */

params.ncbi_api_key = 'Insert your key here'
params.accessions = ['ERR908507', 'ERR908506']

process FASTQC {
    input:
    tuple val(sample_id), path(reads_file)

    output:
    path("fastqc_${sample_id}_logs")

    script:
    """
    mkdir -p fastqc_${sample_id}_logs
    fastqc -o fastqc_${sample_id}_logs -f fastq -q ${reads_file}
    """
}

workflow {
    reads = channel.fromSRA(params.accessions, apiKey: params.ncbi_api_key)
    fastqc (reads)
}
*/
```


## Input declaration

The `val` qualifier receives data of any type as input, it is accessed in the script with the input name

```groovy

num = channel.of(1, 2, 3)

process BASICEXAMPLE{
    debug true

    input:
    val x

    script:
    // input is called by name, here is `x`
    """
    echo process job $x
    """
}

/*  The `path` qualifier  allows the handling of file values.
    It accepts one or more files, and the use of wildcard symbols.
    We can access it on the script using the assigned `name` or a variable reference. */

reads = Channel.fromPath ('../data/ggal/*.fq')
reads2 = channel.fromPath('../data/reads/*.fq.gz')
exercise_ch = channel.fromPath('../data/ggal/*_1.fq')


process FOO {
    debug true // <- prints to std output for debugging 

    input:
    path 'sample.fastq' // <- with name
    path sample // <- variable reference

    script:
    """
    ls sample.fastq
    ls $sample
    """
}

process FOO2 {
    debug true 

    input:
    path sample

    script:
        // expects multiple files 
    """
    ls -lha $sample
    """

}

process EXER1 {
    tag "Concat all files" // <- debug true would not work as no stdout is produced

    input:
    path exer_file

    output:
    path 'top_10_lines'

    script:
    """
    cat $exer_file > merged_file
    head -n 20 merged_file > top_10_lines
    """
}

    /*
    workflow{
        // myrun = BASICEXAMPLE(num)
        // result = FOO(reads, reads2)
        // FOO2(reads2.collect()) // <- collect merges all files from path input
        concat_ch = EXER1(exercise_ch.collect())
        concat_ch.view()
    }
    */

/*  Combine input channels. When one of the channels is empty, the process is 
    terminated, even if a process has more values. */

ch1 = channel.of(1, 2, 3)
ch2 = channel.of('a', 'b', 'c')
    // channels with different number of elements
input1 = channel.of(1, 2)
input2 = channel.of('a', 'b', 'c', 'd', 'e')


process FOO3 {
    debug true 

    input:
    val x
    val y

    script:
        // x and y will be assigned in the order they are specified in workflow
    """
    echo $x and $y
    """
}

process DIFFERENT_CARDINALITY {
    debug true 

    input:
    val 'first_input'
    val 'second_input'

    script:
    """
    echo cardinality: ${first_input} and ${second_input}
    """
}

// Channel.value() can be recycled several times as in the following process

input3 = channel.value(90)

process CHANNEL_VALUE_RECYCLE {
    debug true
    
    input:
    val x
    val y

    script:
    """
    echo recycle: $x and $y
    """
}

    /*
    workflow {
        FOO3(ch1, ch2) // <- ch1 assigned to $x and ch2 to $y
        DIFFERENT_CARDINALITY(input1, input2) // <- different number of values
        CHANNEL_VALUE_RECYCLE(input3, input2)
    }
    */

// Exercise to create a process that repeats a value channel

transcriptome_file = "$baseDir/../data/ggal/transcriptome.fa"
readsEx_ch = channel.fromPath('../data/ggal/*_1.fq')

process EXER2 {
    debug true

    input:
    path transcriptome
    path read_1

    output:
    path result

    script:
    """
    echo salmon -q -r1 $read_1 -t $transcriptome > result
    """
}
```
### Input repeaters

The 'each' qualifier lets you repeat a process for each item in a collection. 
Can be two different methods or different training values. 

```groovy
sequences = Channel.fromPath('../data/prots/*.tfa')
methods = ['regular', 'espresso', 'psicoffee']

process ALIGNSEQUENCES {
    debug true

    input:
    path 'seq'
    each 'mode' // <- each qualifier

    script:
    """
    echo t_coffee -in ${seq} -mode ${mode}
    """
}

    /*
    workflow {
            // repeat value channel
        concat_ch = EXER2(transcriptome_file, readsEx_ch)
        concat_ch.view()
            // apply different values using each qualifier
        ALIGNSEQUENCES(sequences, methods)
    }
    */

reads_ex3 = Channel.fromPath('../data/ggal/*_1.fq')
aligners = ['salmon', 'kallisto']
transcriptome_file = "$baseDir/../data/ggal/transcriptome.fa"

process EXER3 {
    debug true
    tag "Repeat aligners"

    input:
    path reads
    path transcriptome
    each tool

    output:
    path 'alignment_results'

    script:
    """
    echo $tool -r $reads -t $transcriptome > alignment_results
    """
}

workflow {
    result_ch = EXER3 (reads_ex3, transcriptome_file, aligners)
    result_ch.view { "To run: ${it.text}" }
}
```

### Tuple qualifier 

Tuples are non-modifiable groups of variables. They can also be used as inputs for nextflow, specifying each of the qualifier and names

```groovy

reads_ch = Channel.fromFilePairs('../data/ggal/*_{1,2}.fq')

process FOO {
    tag "processing $sample_id"
    debug true 

    input:
    tuple val(sample_id), path(reads)

    output:
    // outputs tuple of a value sample id and a file sample_bam
    tuple val(sample_id), path('sample.bam')

    script:
    """
    echo your_command_here -prefix $sample_id --r1 ${reads[0]} --r2 ${reads[1]} > sample.bam
    """
}

// Exercise naming sample_bam according to sample_id

process EXER1 {
    tag "processing $sample_id"
    debug true 

    input:
    tuple val(sample_id), path(reads)

    output:
    // outputs tuple of a value sample id and a file sample_bam
    tuple val(sample_id), path("${sample_id}.bam")
        // substitution in output assignment requires double quotes

    script:
    """
    echo your_command_here -prefix $sample_id --r1 ${reads[0]} --r2 ${reads[1]} > ${sample_id}.bam
    """
}

workflow {
    // bam_ch = FOO(reads_ch)
    // bam_ch.view { it[1].text }\
        // exercise results
    exer_results = EXER1(reads_ch)
    exer_results.view { "Name: ${it[1].name} -- Content: ${it[1].text}" }
        // To call operators inside double quotes, we use curly braces {}
}
```


## Directives

A **directive** declaration defines optional settings that 
determine the execution of the process. They do not affect the 
semantics of the task itself (input/output/script)

A **directive** declaration is done before 'input' and 'output' declaration.
`tag` is a common directive that we have used so far. 

They are commonly used to assign the computing resources or
to configure other parameters. 

**'publishDir'** directive specifies where to store files for long-term use. 
By default, all files are kept inside the 'work/' folder. 

```groovy

params.outdir = 'my_results'
params.prot2 = "$projectDir/../data/prots/*.tfa"
proteins = Channel.fromPath(params.prot2)

process BLASTSEQ{
    publishDir "$params.outdir/bam_files", mode: 'copy'
        // mode: copy leaves a backup inside the 'work/' folder

    input:
    path fasta

    output:
    path ('*.txt')

    script:
    """
    echo blastp -query $fasta -db nr > ${fasta}_result.txt
    """
}

// To manage subdirectories, we can specify matching patterns.

params.reads = "$projectDir/../data/reads/*_{1,2}.fq.gz"
params.outdir2 = 'my_results'
samples_ch = Channel.fromFilePairs(params.reads, flat:true)

process FOO{
        // creates a subfolder with the sampleId variable
    publishDir "$params.outdir2/$sampleId/", pattern: '*.fq'
        // then it puts the counts and outlook results into subfolders of sampleId
    publishDir "$params.outdir2/$sampleId/counts", pattern: '*_counts.txt'
    publishDir "$params.outdir2/$sampleId/outlooks", pattern: '*_outlooks.txt'

    input:
    tuple val(sampleId), path('sample1.fq.gz'), path('sample2.fq.gz')

    output:
    path "*"

    when:
    params.outdir != "$projectDir"

    script:
    """
    < sample1.fq.gz zcat > sample1.fq
    < sample2.fq.gz zcat > sample2.fq

    awk '{S++}END{print s/4}' sample1.fq > sample1_counts.txt
    awk '{S++}END{print s/4}' sample2.fq > sample2_counts.txt

    head -n 50 sample1.fq > sample1_outlook.txt
    head -n 50 sample1.fq > sample1_outlook.txt
    """
}


workflow {
    blast_ch = BLASTSEQ(proteins)
    blast_ch.view { "${it.baseName} contains: ${it.text}"}
        // when applies a function/operator, use {}
    out_ch = FOO(samples_ch)
}
```

## When declaration

The `when` declaration defines a condition that must be verified
in order to run the script declaration. 

It is somehow an **_'if statement'_** and it must turn out in a boolean
false or true result. 

To evaluate a regular expression use `=~`

```groovy

params.dbtype = 'nr'
params.prot = "$projectDir/../data/prots/*.tfa"
proteins = Channel.fromPath(params.prot)

process FIND {
    tag "Processing ${fasta.baseName} file"
    debug true

    input:
    path fasta
    val type

    output:
    path "${fasta.baseName}.txt"

    when:
    fasta.name =~ /^BB11.*/ && type == 'nr'
        // evaluates if fasta.name contains string
        // && (and) evaluates if type is 'nr' 

    script:
    """
    echo blastp -query $fasta -db ${type} > ${fasta.baseName}.txt
    """
}
        /*
        workflow {
            result = FIND(proteins, params.dbtype)
            result.view {it.text}
        }
        */
```

## Operators

Operators are something like methods in python. They can be used to transform a channel's emission, or apply rules. 

```groovy
//  randomSample() takes a sample of the specified 'n'

Channel
    .of(1..1000)
    .randomSample(12, 439) // <- second value is seed for reproducibility
    //.view()

//  unique() removes duplicates from the channel when emitted

Channel
    .of (1, 2, 3, 6, 6, 6, 7, 7, 8, 1, 4)
    .unique { it % 4 } // <- defines characteristics of uniqueness, picks five only
    //.unique()
    //.view()

//  collect() operator obtains all items emitted by a channel into a list and returns 
//  the list as a single emission. 

Channel
    .fromPath("$projectDir/../data/ggal/*.fq")
    .collect()
    //.view()

Channel
    .of('hello', 'oi', 'hola')
    .collect { it.length() } // <- returns length of each emission
    //.view()

Channel
    .of(['hello', 'hola'], ['oi', 'bonjour', 'hi'], ['konnichiwa', 'namaste', 'ciao'] )
    .collect(flat: true) // <- puts everything in the same list
    //.view()

/*  concat() concatenates the items emitted by a channel in the same order
    they are in the operator arguments. */

a_ch = Channel.of('a', 'b', 'c')
b_ch = Channel.of(1, 2, 3)
c_ch = Channel.of('p', 'q')

    //c_ch.concat(b_ch, a_ch).view() // <- emits c completely, then appends c, and then a_ch

/*  join() creates a new channel that joins the contents of two channels
    according to a matching key. By default it is the first element in each
    emitted item. */

left_ch = channel.of (['X', 1], ['Y', 2], ['Z', 3], ['P', 7])
right_ch = Channel.of(['Z', 6], ['Y', 5], ['X', 2])

    /*
    left_ch.join(right_ch).view()
        // merges based on first element emmited each time, in this case the letter
    left_ch.join(right_ch, by: 1, remainder: true).view() 
        // <- merge by item[1] or second one
        // <- remainder emits unmatched pairs
    */

// The mix() operator combines the items emitted by two or more channels into a single one

c1 = Channel.of( 1, 2, 3 )
c2 = Channel.of( 'a', 'b' )
c3 = Channel.of( 'z' )

    // c1.mix(c2,c3)
        // .subscribe onNext: { println it }, onComplete: { println 'Done' }
        // <- emits in any order
        // <- subscribe specifies a function after every emission

//  The map() operator applies a function of your choosing to every emitted item.
//  The function is expressed with a closure inside curly brackets {}

Channel
    .of(1, 2, 3, 4, 5)
    .map {it * it}
    .subscribe onNext: {println it}, onComplete: { println "Done." }

Channel
    .fromPath('../data/ggal/*_1.fq')
        // can be used to emit tuples too
    .map { pair -> [pair.getBaseName(), pair] }
    .view { name, pair -> "> $name: $pair"}
```

### SplitCsv operator

```groovy
// splitCsv() is an operator similar to splitText() but for comma
// separated values. 

channel
    .fromPath('../data/meta/patients_1.csv')
    .splitCsv()
    .view  { row -> "${row[0]}, ${row[3]}"} // <- row is a list object

    // if file has header, it can be specified and used to call values by column
channel
    .fromPath('../data/meta/patients_1.csv')
    .splitCsv(header:true)
    .view  { row -> "${row.patient_id}, ${row.num_samples}"} // <- row is a list object

    // can also provide custom header names
channel
    .fromPath('../data/meta/patients_1.csv')
    .splitCsv(header: ['col1', 'col2', 'col3', 'col4', 'col5'])
    .view  { row -> "${row.col1}, ${row.col4}, ${row.col5}"} 

    // multiple .csv files can be processed simultaneously
channel
    .fromPath('../data/meta/patients_*.csv') // <- matches a glob pattern
    .splitCsv(header: true)
    .view  { row -> "${row.patient_id}\t${row.num_samples}"} // <- \t is a tab separator

    // can be used without channel definition
def f = file('../data/meta/patients_1.csv') // <- expects a single file
    def lines = f.splitCsv()
    for ( List row : lines) {
        log.info "${row[0]} -- ${row[2]}"
    }

// tsv files can be parsed using .splitCsv() too with the option sep:'\t'
channel
    .fromPath('../data/meta/regions.tsv', checkIfExists:true)
    // use 'sep' as an option to parse TAB separated files
    .splitCsv(sep:'\t', skip:1)
    .view { row -> "$row[0]" }
```

## Modularization 

Modules are used in a similar way to python in Nextflow's DSL2 (Domain Specific Langugage).

You can import processes created in a different script to built your main pipeline. Relative paths are usually specified to keep it portable. 

You can specify a new name for the process, or import it twice if necessary (alias required for latter)

        include {PROCESS as ALIAS}  from './module.nf'
    
For instance, to import multiple modules from the 'hello.nf' script, we could use:

        include {SPLITLETTERS  } from './hello.nf'
        include {CONVERTTOUPPER; REVERSEORDER} from './hello.nf'

```groovy

process SPLITLETTERS {
    input:
    val x

    output:
    path 'chunk_*'

    """
    printf '$x' | split -b 6 - chunk_
    """
}

process CONVERTTOUPPER {
    input:
    path y

    output:
    stdout emit:upper
        // the 'emit' statement defines name to use in channel scope
    
    """
    cat $y | tr '[a-z]' '[A-Z]'
    """
}

process REVERSEORDER {
    debug true 

    input:
    path x

    output:
    stdout

    script:
    """
    rev $x
    """
}

/*  We can also define custom functions (parameters for Groovy) in our module script.
    This ones can be easily invoked in another script as if they were processes. */

params.foo = "Hola"
params.bar = "Mundo!"

def START_RUN() {
    println "$params.foo $params.bar"
}

```

## Workflow declaration

```groovy
#!/usr/bin/env nextflow

params.greeting = "Hello World!"

include { SPLITLETTERS } from './modules.nf'
include { CONVERTTOUPPER } from './modules.nf'

include {  START_RUN } from './modules.nf' addParams(foo: 'Óla')
    // <- define new parameter when importing

workflow my_pipeline {
    
    log.info """
        \
        WORKFLOW.NF in Nextflow $nextflow.version
        =========================================
        RunName         : $workflow.runName
        Launch Dir      : $workflow.launchDir
        Work Dir        : $workflow.workDir
        Start time      : $workflow.start
        """
        .stripIndent()
            // add metadata from nextflow and workflow

    // run imported function
    START_RUN()

    // creates a named pipeline
    SPLITLETTERS(Channel.of(params.greeting))
    CONVERTTOUPPER(SPLITLETTERS.out.flatten())
    CONVERTTOUPPER.out.upper.view{ it }
}

workflow take_pipeline {
        // <- named pipeline
    take:
    greeting
        // <- declares one or more input channels

    main:
        // <- if take/emit is used, pipeline is declared under 'main'
    SPLITLETTERS(greeting)
    CONVERTTOUPPER(SPLITLETTERS.out.flatten())
    CONVERTTOUPPER.out.upper.view {it}
}

workflow emit_pipeline {
    take:
    greeting

    main:
        // <- if take/emit are used, pipeline is declared under 'main'
    SPLITLETTERS(greeting)
    CONVERTTOUPPER(SPLITLETTERS.out.flatten())

    emit:
    my_data = CONVERTTOUPPER.out.upper
        // assigns identifier name to outputs
}

workflow unnamed_emit_pipeline {
    take:
    greeting

    main:
    SPLITLETTERS(greeting)
    CONVERTTOUPPER(SPLITLETTERS.out.flatten())

    emit:
    CONVERTTOUPPER.out.upper // <- provides output as emit_pipeline.out
}


workflow{
    my_pipeline()
        // A named workflow can be invoked in a latter workflow definition
    take_pipeline(Channel.of(params.greeting))
        // In named workflows, processes can be repeated
        // Using take, we can specify the input(s) while calling the workflow
    emit_pipeline(Channel.of(params.greeting))
    emit_pipeline.out.my_data.view()
        // calling out named ('emit') output
    unnamed_emit_pipeline(Channel.of(params.greeting))
    unnamed_emit_pipeline.out.view()
}
```

## Nextflow config

The config file lets you decouple the workflow implementation from the configuration setting. 

By default, Nextflow looks for the 'nextflow.config' file in the current directory
and the script base directory. It also looks for the `$HOME/.nextflow/config`.
When multiple of these files exists, their information is merged before execution
of the pipeline.

The configuration file can also be specified using the option `-c <config.file>`

```groovy 

params.foo = 'óla' 
params.bar = 'Mundo!' 
    // <- params in config file override the ones in other scripts

env.ALPHA = 'some value'
env.BETA = "$HOME/some/path" 
    // <- the 'env' scope defines variables to be exported into the execution environment
```
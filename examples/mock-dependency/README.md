# Mock dependency

## Problem 

You need to synchronize the execution of two processes 
for which there isn't a direct input-output relationship, 
so that process `bar` is executed only after the 
completion of process `foo`.  

## Solution

Add to the outputs of process `foo` a channel producing 
a _flag_ value. 

Then use this channel as input for process `bar` to trigger 
its execution when the other process completes.

## Code 

        Channel
            .fromPath('.data/reads/*.fq.gz')
            .set{ reads_ch }

        process foo {
            output: 
            val true into done_ch

            script:
            """
            your_command_here
            """
        }

        process bar {
            input: 
            val flag from done_ch
            file fq from reads_ch

            script:
            """
            other_commad_here --reads $fq
            """
        }


## Run it

Run the example using this command:


    nextflow run patterns/mock-dependency.nf

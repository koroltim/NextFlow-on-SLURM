# Process per CSV record

## Problem 

You need to execute a task for each record in one or more CSV files.

## Solution 

Read the CSV file line-by-line using the https://www.nextflow.io/docs/latest/operator.html#splitcsv[splitCsv] operator, then use the https://www.nextflow.io/docs/latest/operator.html#map[map] operator to return a tuple with the required field for each line and convert any string path to a file path object using the `file` function.
Finally use the resulting channel as input for the process. 

## Code

Given the file `index.csv` with the following content: 

[%header,format=csv]
|===
sampleId,read1,read2
FC816RLABXX,reads/110101_I315_FC816RLABXX_L1_HUMrutRGXDIAAPE_1.fq.gz,reads/110101_I315_FC816RLABXX_L1_HUMrutRGXDIAAPE_2.fq.gz
FC812MWABXX,reads/110105_I186_FC812MWABXX_L8_HUMrutRGVDIABPE_1.fq.gz,reads/110105_I186_FC812MWABXX_L8_HUMrutRGVDIABPE_2.fq.gz
FC81DE8ABXX,reads/110121_I288_FC81DE8ABXX_L3_HUMrutRGXDIAAPE_1.fq.gz,reads/110121_I288_FC81DE8ABXX_L3_HUMrutRGXDIAAPE_2.fq.gz
FC81DB5ABXX,reads/110122_I329_FC81DB5ABXX_L6_HUMrutRGVDIAAPE_1.fq.gz,reads/110122_I329_FC81DB5ABXX_L6_HUMrutRGVDIAAPE_2.fq.gz
FC819P0ABXX,reads/110128_I481_FC819P0ABXX_L5_HUMrutRGWDIAAPE_1.fq.gz,reads/110128_I481_FC819P0ABXX_L5_HUMrutRGWDIAAPE_2.fq.gz
|===

This snippet parses the file and executes a process for each line:

params.index = 'index.csv'

Channel
    .fromPath(params.index)
    .splitCsv(header:true)
    .map{ row-> tuple(row.sampleId, file(row.read1), file(row.read2)) }
    .set { samples_ch }

process foo {
    input:
    set sampleId, file(read1), file(read2) from samples_ch

    script:
    """
    echo your_command --sample $sampleId --reads $read1 $read2
    """
}


Note: relative paths are resolved by the `file` function against the execution directory. 
In a real use case prefer absolute file paths.

## Run it

Use the the following command to execute the example:


        nextflow run patterns/process-per-csv-record.nf


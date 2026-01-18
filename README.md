# salmonella-enterica-genome-assembly
Background and workflow detailing the steps on assembling the whole genome of the bacteria Salmonella enterica.

## Introduction
Whole-genome sequencing (WGS) has contributed much of our understanding of microbial pathogens. A primary goal of WGS is to computationally reconstruct smaller and fragmented segments of DNA into, ideally, a closed circular genome. Subsequently, this allows analysis goals such as alignment to the reference genome and variant calling to identify genetic differences such as single nucleotide polymorphisms (SNPs) and insertions or deletions (indels) (Paranthaman et al., 2021). Recent advancements in Oxford Nanopore Technologies (ONT) long-read sequencing have expanded the WGS and assembly toolbox. However, challenges remain due to several bacterial species containing many repetitive elements that make accurate assembly difficult (R. R. Wick et al., 2023). Additionally, long-read sequencing has historically exhibited lower per-base quality and is more expensive compared to short-read sequencing technologies, but the gap is closing recently (Goodwin et al., 2016; Iyer et al., 2024). The main goals of this analysis are to assemble raw reads from a Salmonella enterica isolate into a consensus genome, align it to a reference genome, perform variant calling, and visualize.

## Comparing Bioinformatics Tools
Many innovative computational tools have been developed, and new ones continue to emerge. It is important to understand advantages and limitations and to pick appropriate methodologies that produce high-quality and reproducible genome assemblies. SeqKit (Shen et al., 2016a) and Filtlong (R. R. Wick, 2021) are commonly used packages for quality control (QC). SeqKit performs general FASTQ/A manipulations and filtration such as length size and excels at file handling while Filtlong is more optimized for long-read sequencing data (R. R. Wick et al., 2023). For QC visualization, NanoPlot’s key advantages include summary statistics and plots allowing to visually assess quality of long-read sequences (De Coster et al., 2018). Flye (Kolmogorov et al., 2019) and Canu (Koren et al., 2017) are some widely used genome assemblers. Canu generates more accurate assemblies with Pacific Bioscience (PacBio) sequencing but has longer computational times while Flye deals with repetitive genomic repeats more effectively but requires more RAM usage (R. R. Wick & Holt, 2019). Medaka (nanoporetech, 2019) and Nanopolish (Loman et al., 2015) are several programs used for polishing genomes which help reduce assembly errors. Medaka was reported to have higher accuracy than Nanopolish and benefits from picking the appropriate ONT and basecaller model  (R. R. Wick et al., 2023).  For variant calling, BCFtools (Li, 2011) and Medaka (nanoporetech, 2019) are commonly used. When comparing both methods, Medaka had higher variant calling accuracy such as indels and is better suited for long-reads when compared to BCFtools (Hall et al., 2024). For visualizations, many programs exists but Integrative Genomics Viewer (IGV) stands out due to its versatility on different forms such as desktop and its handling of genomic data (Robinson et al., 2011).

## Proposed Methodology
Based on previously evaluated advantages and disadvantages, Nanoplot, Filtlong, Flye, Medaka, and IGV will be selected for this analysis. Raw FASTQ files were generated using ONT sequencer employing R10 chemistry, with expected accuracy scores of ≥Q20. FASTQ files of S. enterica isolate (SRR32410565) were downloaded from NCBI’s Sequence Read Archives (https://trace.ncbi.nlm.nih.gov/Traces/?run=SRR32410565). All bioinformatics tools, except for IGV, were installed via Bioconda on Bash shell environment because of its easy installation process and management of bioinformatics tools. IGV was installed directly on desktop (https://igv.org/doc/desktop/#DownloadPage/) (Robinson et al., 2011). 

## Data preprocessing, filtering, and assembly
The quality of raw reads and filtered reads will be assessed using Nanoplot v1.46.2 (De Coster et al., 2018) from Nanopack collection via fastq input parameter “--fastq” to generate summary statistics and quality visualizations. Filtlong v0.3.1 with parameters “--min_length 1000”, “--keep_percent 90”, and “-q 20” will be applied to remove short reads, poor quality reads, and mean quality below 20. While a quality score of Q20 represents 99% base-calling accuracy, there is a 1 in 100 chance of an incorrect base call. In addition, a higher quality score may impact plasmid recovery since short reads may be removed (R. R. Wick et al., 2023).Genome assembly will be conducted with Flye v2.9.6 (Kolmogorov et al., 2019) with five threads “-t 5”, parameter “--genome-size” will be set to 5m to reflect the known S. enterica genome of around 4.8 Mb, “--asm-coverage 100” for disjointing assembly,  and “--nano-hq” as recommended by Flye’s usage manual for R10 chemistry. Medaka v2.1.1 (nanoporetech, 2019) will be employed for assembly polishing and model parameter set to “r1041_e82_400bps_sup_v5.0.0”. 

## Alignment, Variant Calling, and Visualization
Minimap2 v2.30 (Li, 2018) will be used with options “-ax map-ont” to align the consensus genome to the S. enterica reference genome (RefSeq GCF_000006945.2) and output to Sequence Alignment/Map (SAM) file. Subsequently, SAMtools v1.22.1 (Li et al., 2009) will be applied to convert the resulting SAM files to sorted and indexed Binary Alignment/Map (BAM) files. Medaka v2.1.1 (nanoporetech, 2019) will be employed for variant calling using “medaka_variant”. The genome browser, IGV (Robinson et al., 2011, 2017), will be deployed to visualize variants with inputs from the reference genome, aligned BAM files, and variant call format (vcf) files.


## References

De Coster, W., D’Hert, S., Schultz, D. T., Cruts, M., & Van Broeckhoven, C. (2018). NanoPack: visualizing and processing long-read sequencing data. Bioinformatics, 34(15), 2666–2669. https://doi.org/10.1093/bioinformatics/bty149

Goodwin, S., McPherson, J. D., & McCombie, W. R. (2016). Coming of age: ten years of next-generation sequencing technologies. Nature Reviews Genetics, 17(6), 333–351. https://doi.org/10.1038/nrg.2016.49

Hall, M. B., Wick, R. R., Judd, L. M., Nguyen, A. N., Steinig, E. J., Xie, O., Davies, M., Seemann, T., Stinear, T. P., & Coin, L. (2024). Benchmarking reveals superiority of deep learning variant callers on bacterial nanopore sequence data. ELife, 13. https://doi.org/10.7554/eLife.98300

Iyer, S. V, Goodwin, S., & McCombie, W. R. (2024). Leveraging the power of long reads for targeted sequencing. Genome Research, 34(11), 1701–1718. https://doi.org/10.1101/gr.279168.124

Kolmogorov, M., Yuan, J., Lin, Y., & Pevzner, P. A. (2019). Assembly of long, error-prone reads using repeat graphs. Nature Biotechnology, 37(5), 540–546. https://doi.org/10.1038/s41587-019-0072-8 <br />

Koren, S., Walenz, B. P., Berlin, K., Miller, J. R., Bergman, N. H., & Phillippy, A. M. (2017). Canu: scalable and accurate long-read assembly via adaptive k-mer weighting and repeat separation. Genome Research, 27(5), 722–736. https://doi.org/10.1101/gr.215087.116

Li, H. (2011). A statistical framework for SNP calling, mutation discovery, association mapping and population genetical parameter estimation from sequencing data. Bioinformatics (Oxford, England), 27(21), 2987–2993. https://doi.org/10.1093/bioinformatics/btr509

Li, H. (2018). Minimap2: pairwise alignment for nucleotide sequences. Bioinformatics, 34(18), 3094–3100. https://doi.org/10.1093/bioinformatics/bty191

Li, H., Handsaker, B., Wysoker, A., Fennell, T., Ruan, J., Homer, N., Marth, G., Abecasis, G., Durbin, R., & Subgroup, 1000 Genome Project Data Processing. (2009). The Sequence Alignment/Map format and SAMtools. Bioinformatics, 25(16), 2078–2079. https://doi.org/10.1093/bioinformatics/btp352

Loman, N. J., Quick, J., & Simpson, J. T. (2015). A complete bacterial genome assembled de novo using only nanopore sequencing data. Nature Methods, 12(8), 733–735. https://doi.org/10.1038/nmeth.3444

Paranthaman, K., Mook, P., Curtis, D., Evans, E.-W., Crawley-Boevey, E., Dabke, G., Carroll, K., McCormick, J., Dallman, T. J., & Crook, P. (2021). Development and evaluation of an outbreak surveillance system integrating whole genome sequencing data for non-typhoidal Salmonella in London and South East of England, 2016–17. Epidemiology and Infection, 149, e164. https://doi.org/10.1017/S0950268821001400

Robinson, J. T., Thorvaldsdóttir, H., Wenger, A. M., Zehir, A., & Mesirov, J. P. (2017). Variant Review with the Integrative Genomics Viewer. Cancer Research, 77(21), e31–e34. https://doi.org/10.1158/0008-5472.CAN-17-0337

Robinson, J. T., Thorvaldsdóttir, H., Winckler, W., Guttman, M., Lander, E. S., Getz, G., & Mesirov, J. P. (2011). Integrative genomics viewer. Nature Biotechnology, 29(1), 24–26. https://doi.org/10.1038/nbt.1754

Shen, W., Le, S., Li, Y., & Hu, F. (2016a). SeqKit: A Cross-Platform and Ultrafast Toolkit for FASTA/Q File Manipulation. PLOS ONE, 11(10), e0163962-. https://doi.org/10.1371/journal.pone.0163962

Wick, R. R. (2021). Filtlong. https://github.com/rrwick/Filtlong

Wick, R. R., & Holt, K. E. (2019). Benchmarking of long-read assemblers for prokaryote whole genome      sequencing. F1000Research, 8, 2138. https://doi.org/10.12688/f1000research.21782.3

Wick, R. R., Judd, L. M., & Holt, K. E. (2023). Assembling the perfect bacterial genome using Oxford Nanopore and Illumina sequencing. PLOS Computational Biology, 19(3), e1010905-. https://doi.org/10.1371/journal.pcbi.1010905











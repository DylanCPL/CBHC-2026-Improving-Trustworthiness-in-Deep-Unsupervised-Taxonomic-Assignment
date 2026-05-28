# CBHC-2026-Improving-Trustworthiness-in-Deep-Unsupervised-Taxonomic-Assignment

Additional information for the poster presentation "Improving Trustworthiness in Deep Unsupervised Taxonomic Assignment", presented at CBHC 2026 by Dylan Lewis. Note that the poster design was inspired by the BetterPoster template, introduced by Mike Morrison in 2019. 

# Descriptions and implementation details of DeLUCS, *i*DeLUCS, and *i*DeLUCS-F

The DeLUCS pipeline [6] uses DNA sequences and their identifiers (taxonomic ascension IDs) as input and, in
evaluation mode, the associated class labels. Sequences are filtered to remove ambiguous bases, retaining only A, C,
G, and T nucleotides. For each sequence, *n* mimics (created through mutations on the original associated sequence) are created, normalized 6-mer counts are obtained for that sequence and its associated *n* mimics, creating *n* original-and-
mimic pairs. Following normalization, an ensemble of ten neural networks are trained using IIC loss on each original-
and-mimic-pair. Each network is trained for 150 epochs, and Gaussian noise is introduced to the networks’ parameters.
After the networks make their predictions, the predictions of networks 2–10 are mapped to the predictions of network 1
using the Hungarian algorithm, and the aligned predictions are combined using a majority vote. In deployment, the
mapping between each sequence identifier and cluster ID is returned. In evaluation mode, the Hungarian algorithm
is used again to find the optimal mapping between the majority vote predictions and the true classes, in order to then
return the clustering accuracy.

The DeLUCS pipeline, as retrieved from the GitHub link given in the DeLUCS publication [6], required several
modifications prior to its use in this study. Most notably, mapping each network’s predictions to the true class labels
using the Hungarian algorithm before combining them via majority vote produced an overly optimistic estimate
of clustering accuracy. Correcting this yields lower performance than reported in the original DeLUCS
study [6]. Additionally, adjustments to the source code were required to ensure that
only *n* original-and-mimic pairs were created (rather than 3(*n* − 2)), and that the mimic sequences were not overwritten
by the original sequences. The *i*DeLUCS study [8] evaluates performance using the original DeLUCS pipeline, however the DeLUCS pipeline used is unchanged from the DeLUCS study [6].

The iDeLUCS pipeline [8] similarly takes as input a dataset of DNA sequences and sequence identifiers (and
when in evaluation mode, also uses associated class labels). All letters representing ambiguity in bases, or gaps, are
converted to the letter ‘N’, and any ‘U’ is converted to ‘T’. Uracil-to-thymine nucleotide substitutions were introduced
into the original *i*DeLUCS framework to address instances of DNA damage present in the *i*DeLUCS datasets, but are
not relevant for the datasets analyzed in this study. Afterwards, only the letters A, C, T, G, and N are retained.
The data augmentation process in *i*DeLUCS transforms all natural DNA sequences prior to training; thus, rather
than training the neural networks with original/augmented pairs (as was the case with DeLUCS), instead they are trained
with augmented/augmented pairs. Baseline, original-like sequence representations are first computed by applying a
combined mutational model inducing transition mutations at a rate of 1.0% and transversion mutations at a rate of
0.5%. The first augmentation step then generates a mutated version of each original DNA sequence using a transition
mutation rate of 1.0%, while the second independently generates a separate mutated version by applying a transversion
mutation rate of 0.5% to the original sequence. The remaining *n*–2 augmentations are produced via random replacement
of 20 bases with the ambiguous nucleotide base code N.

For effective representation learning, the final training set comprises multiple pairs of related sequences, in which
a baseline (“original-like”) encoding is coupled with independently generated augmented views, including transition-
based, transversion-based, and noise-induced variants. In contrast, the test set (analogous to supervised learning, the test
set referring to the dataset upon which predictions are made after training) consists solely of baseline representations
without augmentation. Then 6-mers are counted for all sequences, excluding any 6-mers containing the ambiguous
nucleotide ‘N’, and these counts are then normalized.

Five neural networks are trained independently on paired *k*-mer frequency vectors, and their predicted cluster
assignments are aggregated through a feature-based consensus procedure, in which cluster memberships from all
models are encoded and subsequently clustered using k-means++ to produce the final labels. The networks are trained
using a combination of contrastive loss (InfoNCE) applied to latent representations and IIC-based loss applied to cluster
assignment outputs, encouraging consistent predictions between the two sequences of each pair. Unlike DeLUCS,
during training, no Gaussian noise is introduced. In deployment, each sequence is assigned a cluster label, and this
mapping is returned, as well as the latent representation learned by the best-performing network (i.e., the network which best minimized its loss function), enabling for visualization of how the model clustered the sequences, as well
as measuring the effectiveness of the latent space for clustering by use of intrinsic clustering metrics. In evaluation
mode, an optimal mapping between predicted clusters and true classes (e.g., via the Hungarian algorithm) is used to
compute clustering accuracy and other extrinsic metrics.

*i*DeLUCS differs from the original DeLUCS pipeline through ensemble training on augmented-augmented
sequence pairs as opposed to original-augmented pairs, use of N-injection for data augmentation, ambiguous base
resolution, and modified model default parameters (for example, five voters are used instead of ten).
As a fragment-based alternative to *i*DeLUCS, *i*DeLUCS-F seeks to improve model trustworthiness by replacing
synthetic data augmentation and grounding model training in a more biologically relevant technique. In the *i*DeLUCS-F
pipeline, rather than mutated-mutated sequence pairs and mutated-N-injected sequence pairs, original-fragment and
original-N-injected sequence pairs are used. This approach verifies that the model is learning on both natural and
transformed DNA sequences. Fragments are generated using 99% and 99.5% relative sequence lengths, analogous to
the mutation rates of 1% and 0.5% for the mutated sequences in the first and second augmentation pairs.
Rather than replacing N-injection augmentation with further fragments, to effectively investigate *only* the effect
of supplanting mutation-based augmentation with a fragmentation-based approach, the N-injection step was left
unchanged. While N-injection may not generate sequences representative of natural evolutionarily base changes, it does
effectively imitate the introduction of errors and artifacts from genomic reconstruction using sequencing reads [10].

# References
[1] C. C. Thompson, L. Chimetto, R. A. Edwards, J. Swings, E. Stackebrandt, and F. L. Thompson. Microbial genomic taxonomy. BMC Genomics, vol. 14, no. 1, p. 913, 2013, doi: 10.1186/1471-2164-14-913. <br>
[2] G. S. Randhawa, M. P. Soltysiak, H. E. Roz, C. P. de Souza, K. A. Hill, and L. Kari. Machine learning using intrinsic genomic signatures for rapid classification of novel pathogens: COVID-19 case study. PLoS ONE, 15(4):e0232391, 2020. <br>
[3] D. Emerson, L. Agulto, H. Liu, and L. Liu. Identifying characterizing bacteria in an era of genomics proteomics. BioScience, 58(10):925–936, 2008. <br>
[4] J. Avila Cartes, S. Anand, S. Ciccolella, P. Bonizzoni, and G. D. Vedova. Accurate fast clade assignment via deep learning frequency chaos game representation. Oxford University Press GigaScience, 12:giac119, 2022. <br>
[5] J. Rajkumari, P. Katiyar, S. Dheeman, P. Pandey, and D. K. Maheshwari. The changing paradigm of rhizobial taxonomy and its systematic growth upto postgenomic technologies. World J Microbiol Biotechnol, vol. 38, no. 11, p. 206, Nov. 2022, doi: 10.1007/s11274-022-03370-w. <br>
[6] P. M. Arias, F. Alipour, K. A. Hill, and L. Kari. DeLUCS: Deep learning for unsupervised clustering of DNA sequences. PLOS ONE, vol. 17, no. 1, p. e0261531, Jan. 2022, doi: 10.1371/journal.pone.0261531. <br>
[7] W. L. Applequist. A brief review of recent controversies in the taxonomy and nomenclature of sambucus nigra sensu lato. Acta Hortic., no. 1061, pp. 25–33, Jan. 2015, doi: 10.17660/ActaHortic.2015.1061.1. <br>
[8] P. Millan Arias, K. A. Hill, and L. Kari. iDeLUCS: a deep learning interactive tool for alignment-free clustering of DNA sequences. Bioinformatics, vol. 39, no. 9, p. btad508, Sep. 2023, doi: 10.1093/bioinformatics/btad508. <br>
[9] S. Solis-Reyes, M. Avino, A. Poon, and L. Kari. An open-source k-mer based machine learning tool for fast accurate subtyping of HIV-1 genomes. PLoS ONE, 13(11):e0206409, 2018.

Cited here, but not in poster:

[10] D. Laehnemann, A. Borkhardt, and A. C. McHardy, “Denoising dna deep sequencing data—high-throughput
sequencing errors and their correction,” Briefings in Bioinformatics, vol. 17, no. 1, pp. 154–1

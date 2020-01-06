## Version highlights for GATK version 3.4

By Geraldine_VdAuwera

<p>Folks, I’m all out of banter for this one, so let’s go straight to the facts. GATK 3.4 contains a shedload of improvements and bug fixes, including some new functionality that we hope you’ll find useful. The full list is available in the detailed <a rel="nofollow" href="https://www.broadinstitute.org/gatk/guide/version-history">release notes</a>.</p>

<p>None of the recent changes involves any disruption to the Best Practice workflow (I hear some sighs of relief) but you’ll definitely want to check out the tweaks we made to the joint discovery tools (HaplotypeCaller, CombineGVCFs and GenotypeGVCFs), which are rapidly maturing as they log more flight time at Broad and in the wild.</p>

<hr></hr><h3>Key changes to the joint discovery tools</h3>

<p>Let’s start at the very beginning with <strong>HaplotypeCaller</strong> (a very good place to start). On the usability front, we’ve finally given in to the nigh-universal complaint about the required variant indexing arguments (<code class="code codeInline" spellcheck="false">--variant_index_type LINEAR --variant_index_parameter 128000</code>) being obnoxious and a waste of characters. So, tadaa, they are no longer required, <em>as long as you name your output file with the extension <code class="code codeInline" spellcheck="false">.g.vcf</code></em> so that the engine knows what level of compression to use to write the gVCF index (which leads to better performance in downstream tools). We think this naming convention makes a lot of sense anyway, as it’s a great way to distinguish gVCFs from regular VCFs on sight, so we hope most of you will adopt it. That said, we stopped short of making this convention mandatory (for now…) so you don’t have to change all your scripts and conventions if you don’t want to. All that will happen (assuming you still specify the variant index parameters as previously) is that you’ll get a warning in the log telling you that you could use the new convention.</p>

<p>Where we’ve been a bit more dictatorial is that we’ve completely disabled the use of <code class="code codeInline" spellcheck="false">-dcov</code> with HaplotypeCaller because it was causing very buggy behavior due to an unforeseen complication in how different levels of downsampling are applied in HaplotypeCaller. We know that the default setting does the right thing, and there’s almost no legitimate reason to change it, so we’re disabling this for the greater good pending a fix (which may be a long time coming due to the complexity of the code involved).</p>

<p>Next up, <strong>CombineGVCFs</strong>  gets a new option to break up reference blocks at every N sites. The new argument <code class="code codeInline" spellcheck="false">--breakBandsAtMultiplesOf N</code>will ensure that no reference blocks in the combined gVCF span genomic positions that are multiples of N. This is meant to enable scatter-gather parallelization of joint genotyping on whole-genome data, as a workaround to some annoying limitations of the GATK engine that make it unsafe to use <code class="code codeInline" spellcheck="false">-L</code> intervals that might start within the span of a block record. For exome data, joint genotyping can easily be parallelized by scatter-gathering across exome capture target intervals, because we know that there won’t be any hom-ref block records spanning the target interval boundaries. In contrast, in whole-genome data, there is no equivalent predictable termination of block records, so it’s not possible to know up front where it would be safe to set scatter-gather interval start and end points -- until now!</p>

<p>And finally, <strong>GenotypeGVCFs</strong> gets an important bug fix, and a very useful new annotation.</p>

<p>The bug is something that has arisen mostly (though not exclusively) from large cohort studies. What happened is that, when a SNP occurred in sample A at a position that was in the middle of a deletion for sample B, GenotypeGVCFs would emit a homozygous reference genotype for sample B at that position -- which is obviously incorrect. The fix is that now, sample B will be genotyped as having a symbolic <code class="code codeInline" spellcheck="false">&lt;*:DEL&gt;</code> allele representing the deletion.</p>

<p>The new annotation is called <code class="code codeInline" spellcheck="false">RGQ</code> for Reference Genotype Quality. It is a new sample-level annotation that will be added by GenotypeGVCFs to monomorphic sites if you use the <code class="code codeInline" spellcheck="false">-allSites</code> argument to emit non-variant sites to the output VCF. This is obviously super useful for evaluating the level of confidence of those sites called homozygous-reference.</p>

<hr></hr><h3>New RNAseq tool: ASEReadCounter</h3>

<p>This new coverage analysis tool is designed to count read depth in a way that is appropriate for allele-specific expression (ASE) analysis. It counts the number of reads that support the REF allele and the ALT allele, filtering low qual reads and bases and keeping only properly paired reads. The default output format produced by this tool is a structured text file intended to be consumed by the statistical analysis toolkit MAMBA. A paper by Stephane Castel and colleagues describing the complete ASE analysis workflow is available as a <a rel="nofollow" href="http://biorxiv.org/content/early/2015/03/05/016097">preprint on bioarxiv</a>.</p>

<hr></hr><h3>New documentation features: “Common Problems” and “Issue Tracker”</h3>

<p>We’ve added two new documentation resources to the <a rel="nofollow" href="https://www.broadinstitute.org/gatk/guide/">Guide</a>.</p>

<p>One is a new category of documentation articles called <a rel="nofollow" href="https://www.broadinstitute.org/gatk/guide/topic?name=problems">Common Problems</a>, to cover topics that are a specialized subset of FAQs: problems that many users encounter, which are typically due to misunderstandings about input requirements or about the expected behavior of the tools, or complications that arise from certain experimental designs. This category is being actively worked on and we welcome suggestions of additional topics that it should cover.</p>

<p>The second is an <a rel="nofollow" href="https://www.broadinstitute.org/gatk/guide/issue-tracker">Issue Tracker</a> that lists issues that have been reported as well as features or enhancements that have been requested. If you encounter a problem that you think might be a bug (or you have a feature request in mind), you can check this page to see if it’s something we already know about. If you have submitted a bug report, you can use the issue tracker to check whether your issue is in the backlog, in the queue, or is being actively worked on. In future we’ll add some functionality to enable voting on what issues or features should be prioritized, so stay tuned for an announcement on that!</p>
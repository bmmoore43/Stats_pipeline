# Stats_pipeline

1. The first step is to get your matrices
2. Next combine matrix with class file of the two classes you will compare
   
   Run:
         
            python script_combine_exression_matrix.py <classes file> <matrix file>

3. Select matrix type:
    
    A. For binary matrices:
    
    1. Get feature percents and logratios: 
    
    note if the other class has 0 genes, will use 0.000001 
    as an approximation to calculate the logratio
      
            python parse_binary_get_feature_percent_logratio.py <binary matrix file> <gene type of negative class>
            
            python ~john3784/Github/parse_scripts/parse_binary_get_feature_percent_logratio.py SMvsPM_enz_2.0-kinases.excluded-aracyc.txt-binary_matrix_2.0_mod.txt PM
         
    2. Get statistical significance, using fishers exact test. 
    
    First get enrichment table:
      
            python parse_binary_get_enrich_table.py <binary matrix file>
            
            python ~john3784/Github/parse_scripts/parse_binary_get_enrich_table.py SMvsPM_enz_2.0-kinases.excluded-aracyc.txt-binary_matrix_2.0_mod.txt
         
            output= binary_matrix_file_enrichment_table.txt
         
    3. Then do fishers exact test:
    
            python Test_Fisher.py <enrichment table file> 1 # 1 = with q-value, 0 = without
            
            #if you can't run Test_Fisher.py, try running from Alex's script ~seddonal/scripts/6_motif_mapping/Test_Fisher.py
            
            python ~seddonal/scripts/6_motif_mapping/Test_Fisher.py SMvsPM_enz_2.0-kinases.excluded-aracyc.txt-binary_matrix_2.0_mod.txt_enrichment_table.txt 1
        
    4. Make barplot of pvalues: need significant ones
     
     gets significant features and converts qvalue to the -log(qvalue)
     
            python parse_enrichment_get_sig.py <.pqvalue file from fisher's exact>
         
            python ~john3784/Github/parse_scripts/parse_enrichment_get_sig.py SMvsPM_enz_2.0-kinases.excluded-aracyc.txt-binary_matrix_2.0_mod.txt_enrichment_table.txt.fisher.pqvalue
            
            output= .sig_score file
         
     barplot:
     
            R --vanilla --slave --args <dir with .sig_score file> <.sig_score file> < /mnt/home/john3784/Github/R_scripts/barplot_features_s.R
         
            R --vanilla --slave --args /mnt/home/john3784/2-specialized_metab_project/machine-learn_matrices/ SMvsPMclasses_file.2.0.txt-    enzymatic_genes2.0.txt-binary_matrix_2.0_mod.txt_enrichment_table.txt.fisher.pqvalue.sig_score < /mnt/home/john3784/Github/R_scripts/barplot_features_s.R
         
     5. get the logratios for significant features and make barplot:
     
     first combine sig_score file with logratio file:
     
            python script_combine_exression_matrix.py <logratio file> <sig score file>
            
     make barplot of significant logratio:
     
            R --vanilla --slave --args <dir with logratio-sig_score file> <logratio-sig_score file> < /mnt/home/john3784/Github/R_scripts/barplot_logratio_only.R
     
     For enrichment with clusters (ie. GO enrichment):
     
     1. get cluster enrichment
     
            python cluster_enrichment_final.py <file with gene: cluster> <file with either Go term:gene or gene:pathway/class>
            
        returns tableOfEnrichment_file
        
     2. use tableOfEnrichment_file to do fisher exact test
     
            python ~john3784/Github/GO_enrichment/Test_Fisher.py <tableforEnrichment> <0 for pval, 1 for pval and qval>
            
     3. get significant ones only:
     
            python ~john3784/Github/GO_enrichment/parse_significant2.py <.fisher.pqvalue file>
            
     4. get a matrix of significant and non-signif SM and PM genes in a cluster where 0= no SM or PM genes in cluster 1= SM genes (signif or not in cluster), 2= PM genes (significant or not in cluster)
     
            python get_matrix_sigSM-PMclusters.py <.fisher.pqvalue file> <gene:cluster file>
            
            python ~john3784/Github/GO_enrichment/get_matrix_sigSM-PMclusters.py tableforEnrichment_genelist_h100_dev_average.fisher.pqvalue genelist_h100_dev_average.header.txt
     
     For continuous matrix:
     
     1. get statisitical significance of each continous feature between two classes using the wilcox test (mann whitney u)
     
                R --vanilla --slave --args <dir with continuous matrix file> <continous matrix file> < /mnt/home/john3784/Github/R_scripts/wilcox_test.R
            
     2. make boxplots for each continuous feature
     
              R --vanilla --slave --args <dir with continuous matrix file> <continous matrix file> < /mnt/home/john3784/Github/R_scripts/boxplot_features_s.R
            
     For categorical data:
     
     1. These are gene clusters. We want to know if there is a difference between SM and PM in terms of clustering. First we look at cluster size:  
     If you want to know what gene is in what cluster, and how many genes are in a cluster, run this script on the directory with cluster gene files (file with gene, cluster.number):
     
               python parse_categ_get_genenum.py <dir with directories of gene_list files> <file with all genes> <output name>
            
               python parse_categ_get_genenum.py /mnt/home/john3784/2-specialized_metab_project/clustering/ AT_all-genes.txt cluster_gene_num_
            
      2. Then combine with classes file:
     
                for i in cluster_gene_num*; do echo $i; python /mnt/home/john3784/Github/parse_scripts/script_combine_exression_matrix.py /mnt/home/john3784/2-specialized_metab_project/machine-learn_matrices/SMvsPM_classes-enz_2.0-kinases.excluded.txt $i; done
            
     3. Next do wilcox test on file to see if there is significant difference between cluster size
     
                R --vanilla --slave --args <dir with files> <cluster gene number file with class> < /mnt/home/john3784/Github/R_scripts/wilcox_test2.R
            
               for i in SMvsPM_classes-enz_2.0-kinases.excluded.txt-cluster_gene_num_*; do echo $i; R --vanilla --slave --args /mnt/home/john3784/2-specialized_metab_project/clustering/approx_k_kmeans/ $i < /mnt/home/john3784/Github/R_scripts/wilcox_test2.R; done       
      4. Concatenate files:
     
               cat *.wilcoxtest.txt > akk_all_wilcoxtest.txt
     
      5. If you want to know what are the significant clusters for each gene type, first create a table of enrichment for each cluster:
     
               python parse_categ_get_enrichment.py <dir with directories of gene_list files> <classes_file>
            
               python /mnt/home/john3784/Github/parse_scripts/parse_categ_get_enrichment.py /mnt/home/john3784/2-specialized_metab_project/clustering/ SMvsPM_classes-enz_2.0-kinases.excluded.txt
     
     6. Use the fishers exact test on these enrichment tables:
     
             python Test_Fisher.py <enrichment table> <0 or 1, 1 for qvalue>
             python /mnt/home/john3784/Github/parse_scripts/Test_Fisher.py akk1000.stressfc.run5.enrichment_table.txt 1
              
     use a unix loop for multiple tables:
     
            for i in *.enrichment_table.txt; do echo $i; python /mnt/home/john3784/Github/parse_scripts/Test_Fisher.py $i 1; done
     
     
     7. parse enrichment table to get significant clusters and convert qvalue to the -log(qvalue)
     
              python parse_enrichment_get_sig.py <.pqvalue file from fisher's exact>
         
              output= .sig_score file
         
    unix loop:
    
            for i in *.fisher.pqvalue; do echo $i; python /mnt/home/john3784/Github/parse_scripts/parse_enrichment_get_sig.py $i; done
   
   concatenate:
   
            cat *.sig_score > akk_all_sig_score.txt
   barplot:
     
            R --vanilla --slave --args <dir with .sig_score file> <.sig_score file> < /mnt/home/john3784/Github/R_scripts/barplot_features_s.R
          
         

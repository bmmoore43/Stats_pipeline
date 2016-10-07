# Stats_pipeline

1. The first step is to get your matrices
2. Next combine matrix with class file of the two classes you will compare
   
   Run:
         
            python script_combine_exression_matrix.py <classes file> <matrix file>

3. Select matrix type
    
    For binary matrices:
    
    1. Get feature percents and logratios: 
    
    note if the other class has 0 genes, will use 0.000001 
    as an approximation to calculate the logratio
      
            python parse_binary_get_feature_percent_logratio.py <binary matrix file> <gene type of negative class>
         
    2. Get statistical significance, using fishers exact test. 
    
    First get enrichment table:
      
            python parse_binary_get_enrich_table.py <binary matrix file>
         
            output= binary_matrix_file_enrichment_table.txt
         
    Then do fishers exact test:
    
            python Test_Fisher.py <enrichment table file> 1 # 1 = with q-value, 2 = without
        
     3. Make barplot
     
     Of pvalues: need significant ones
     
     gets significant features and converts qvalue to the -log(qvalue)
     
            python parse_enrichment_get_sig.py <.pqvalue file from fisher's exact>
         
            output= .sig_score file
         
     barplot:
     
            R --vanilla --slave --args <dir with .sig_score file> <.sig_score file> < /mnt/home/john3784/Github/R_scripts/barplot_features_s.R
         
            R --vanilla --slave --args /mnt/home/john3784/2-specialized_metab_project/machine-learn_matrices/ SMvsPMclasses_file.2.0.txt-    enzymatic_genes2.0.txt-binary_matrix_2.0_mod.txt_enrichment_table.txt.fisher.pqvalue.sig_score < /mnt/home/john3784/Github/R_scripts/barplot_features_s.R
         
     For continuous matrix:
     
     1. get statisitical significance of each continous feature between two classes using the wilcox test (mann whitney u)
     
            R --vanilla --slave --args <dir with continuous matrix file> <continous matrix file> < /mnt/home/john3784/Github/R_scripts/wilcox_test.R
            
     2. make boxplots for each continuous feature
     
            R --vanilla --slave --args <dir with continuous matrix file> <continous matrix file> < /mnt/home/john3784/Github/R_scripts/boxplot_features_s.R
            
     For categorical data:
     
     1. These are gene clusters. We want to know if there are significant clusters for each gene type. 
     If you want to know what gene is in what cluster, and how many genes are in a cluster, run this script on the directory with cluster gene files (file with gene, cluster.number):
     
            python parse_categ_get_genenum.py <dir with directories of gene_list files> <file with all genes> <output name>
            
            python parse_categ_get_genenum.py /mnt/home/john3784/2-specialized_metab_project/clustering/ AT_all-genes.txt cluster_gene_num_
            
     2. To get the significant ones, create a table of enrichment for each cluster:
     
            python parse_categ_get_enrichment.py <dir with directories of gene_list files> <classes_file>
            
            python /mnt/home/john3784/Github/parse_scripts/parse_categ_get_enrichment.py /mnt/home/john3784/2-specialized_metab_project/clustering/ SMvsPM_classes-enz_2.0-kinases.excluded.txt
     
     3. Use the fishers exact test on these enrichment tables:
     
            python Test_Fisher.py <enrichment table> <0 or 1, 1 for qvalue>
            python /mnt/home/john3784/Github/parse_scripts/Test_Fisher.py akk1000.stressfc.run5.enrichment_table.txt 1
              
     use a unix loop:
     
            
          
         

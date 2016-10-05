# Stats_pipeline

1. The first step is to get your matrices
2. Next combine matrix with class file of the two classes you will compare
   
   Run:
         
         python script_combine_exression_matrix.py <classes file> <matrix file>

3. Select matrix type
    
    For binary matrices:
    
    1. Get feature percents and logratios: 
    
    #note if the other class has 0 genes, will use 0.000001 
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
     
     #gets significant features and converts qvalue to the -log(qvalue)
     
         python parse_enrichment_get_sig.py <.pqvalue file from fisher's exact>
         
         output= .sig_score file
         
     barplot:
     
         
     
          
         

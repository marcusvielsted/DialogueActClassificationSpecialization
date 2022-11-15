********************************************************README********************************************************

___Content of the repository___

                                                                    "140sentimentSmall"
This folder contains a twitter corpus, "140sentimentSmall.csv" used for early corpus exploration. This is not used for the rest of this codebase
Source: https://www.kaggle.com/kazanova/sentiment140




                                                                    "npsCHAT"
This is the directory of our current source domain, NPSCHAT corpus. Within this folder. Readme for the corpus can be found under "nps_chat"

                        "XML Files"
This folder contains the original NPS chat in XML format split among 15 files. Each file emcompasses chat interactions on a specific day
 

                        "POS files"
For each post in the XML files, we have a corresponding file with the POS-tags for all post




                                                                    "RedditConviKit"
                        "RedditDatasetCreation.ipynb"
This file contains the script used to create our Reddit Dataset. Run alle blocks again to create a similar dataset ** HOWEVER, since we have a ".sample" function, it 
it not certain you create the exact same dataset.


                        "RedditCorpusAnnotated.csv"
This file contains our annotated target domain, with each utterance having 3 labels; "MlviLabel","NikwLabel","Label". The first two label-columns corresponds to our individial annotation of a given utterance, 
while the final colum "label" corresponds to label the annotators reached upon resolving any disagreement





                                                                    "Scripts"
                        "Resampling.ipynb"
File containing the creation and exploration of our multinomial Resampling function with corresponding vicualizations
                        
                        "SVM_NPS_Optimized.ipynb"
Contains the script for our optimzed In-domain dialogue act classification model
                       
                        "SVM_Reddit_Baseline.ipynb"
Contains the script for our baseline cross-domain dialogue act classification model

                        "SVM_Reddit_Optimized.ipynb"
Contains the script for our optimzed Cross-domain dialogue act classification model

                                                                   





___HOW To RUN The Code:___
-------------------------------------------------------------

Requirements: Python 3, Jypiter Notebooks, Pandas, Numpy

Under the folder "Scripts" we have our different models. The main model which the corresponding paper refers to, is "SVM_Reddit_Optimized.ipynb" for cross-domain and "SVM_NPS_Optimized.ipynb" for in-domain. For our baseline cross-domain model
you can find it under "SVM_Reddit_Baseline.ipynb", which is the one we mainly compare our optimized cross-domain model against

---Running the optimized Cross-domain model:---



We have structured the codebase into different blocks. We recommend comment out the block regarding cross-validation, as this process takes quite some time
It is important that both the source domain (NPS) and the target domain (Reddit) are located on the same relative path at all times;
-NPS: "../npsChat/XML Files"             # Only point the the folder containing all of our XML files
-Reddit:  "../RedditConviKit/RedditCorpusAnnotated.csv"  

We have a section in the middle of our codebase, which generates an excel file for wordify. As default this is commented out, but 
should you need to generate this file, you can easily just comment these in;
'''''''''''''''''''''''''''''''''
            # # Comment in if you need to create the file
            # df_NPSPre["Text"] = df_NPSPre["Text"].apply(lambda x: custom_Preproccesing(x))
            # df_NPSPre.to_excel("Towordify.xlsx") 
'''''''''''''''''''''''''''''''''

Otherwise you can run all codeblocks sequentially to produce that same results as we have:

-----***-----Changing parameters and feature-----***-----


Changing our multinomial re-sampling alpha:__________________________________________________________________________________
- This is done by changing the parameter giving to our sample_smoothing funciton under the codeblock 
" Cross domain specific Pre-processing for SVM":
'''''''''''''''''''''''''''''''''
            # Re sampleling
            df_NPS = sample_smoothing(df_NPSPre, 0.5)  # Change to any value between 1.0 and 0.0
'''''''''''''''''''''''''''''''''

Changing parameters for our vectorized vocabulary:__________________________________________________________________________
The following featureUnion currently creates of vocabulary based on transformations on our input domains;
'''''''''''''''''''''''''''''''''
            FU = FeatureUnion([
                ("CountVecWord",CountVectorizer(preprocessor=custom_Preproccesing, tokenizer=custom_tokenizer, ngram_range=(1,1), binary = False)),
                ("VecLen",FeaturizerLength()),
                ("VecFirstWord", FeaturizerFirstWord()),
            ])
'''''''''''''''''''''''''''''''''
-The fuction "custom_Preproccesing" is a custom created funtion used to transform any input data. Any changes can be made to function
and it will applied to the input datasets

-the function "custom_tokenizer" contains our current TweetTokenizer. If you need to change tokenizer, you can change it within this function

-The different n-gram ranges can be changed within the "ngram_range=(1,1)". (1,2) create uni- and bigrams, (2,2) only bi grams etc

-if you want a binary representation of the token frequency, you can change "binary" to "True)

-If you want to have character based n-grams, add the parameter (analyzer =‘char_wb’) to the CountVectorizer

-If you want to include our custom stop-words, add the following to the CountVectorizer; (,stop_words="custom_Stop_words",)

-If you want both a character and word based n-gram, add the following codesnippet to the featureUnion
'''''''''''''''''''''''''''''''''
            ("CountVecChar",CountVectorizer(preprocessor=custom_Preproccesing,tokenizer=custom_tokenizer,ngram_range=(2,5), binary = True, analyzer="char_wb")),
'''''''''''''''''''''''''''''''''

-If you want to exclude any custom feature group, these can be commented out from the FeatureUInion pipeline



Changing parameter for our model:________________________________________________________________________________________
If you want to change hyper-parameters for our SVM model, this can be done by changing the values as shown below;
'''''''''''''''''''''''''''''''''
            rbf_reddit = svm.SVC(kernel='rbf', gamma=0.01, C=11).fit(X_vectorized, y)
'''''''''''''''''''''''''''''''''
Here you can change the kernal, gamma and C values.








---Running the optimized in-domain model:---

We have structured the codebase into different blocks. We recommend comment out the block regarding cross-validation, as this process takes quite some time
It is important that both the source domain (NPS) and the target domain (Reddit) are located on the same relative path at all times;
-NPS: "../npsChat/XML Files"             # Only point the the folder containing all of our XML files

you can run all codeblocks sequentially to produce that same results as we have:

-----***-----Changing parameters and feature-----***-----



Changing our multinomial re-sampling alpha:__________________________________________________________________________________
- This is done by changing the parameter giving to our sample_smoothing funciton under the codeblock 
"# Model name: Support Vector Machine (SVM)":
'''''''''''''''''''''''''''''''''
            train_sampled = sample_smoothing(train, 0.8) # Change to any value between 1.0 and 0.0
'''''''''''''''''''''''''''''''''

Changing parameters for our vectorized vocabulary:__________________________________________________________________________
The following featureUnion currently creates of vocabulary based on transformations on our input domains; 
'''''''''''''''''''''''''''''''''
            Featureizer = FeatureUnion([
                ("TdifVecWord", TfidfVectorizer(tokenizer=twitter_Tok, binary=True, ngram_range=(1,1))),
                ("TdifVecChar", TfidfVectorizer(tokenizer=twitter_Tok, binary=True, ngram_range=(2,3), analyzer="char_wb")),
                # ("VecLen",FeaturizerLength()),
                ("VecFirstWord", FeaturizerFirstWord()),
            ])
'''''''''''''''''''''''''''''''''

Same transformations can be made by changing the parameters / commenting out any steps as described above

Changing parameter for our model:________________________________________________________________________________________
If you want to change hyper-parameters for our SVM model, this can be done by changing the values as shown below;
'''''''''''''''''''''''''''''''''
            rbf_nps = svm.SVC(kernel='rbf', gamma='scale', C=4).fit(X_train_vectorized, y_train)
'''''''''''''''''''''''''''''''''
Here you can change the kernal, gamma and C values.










---Running the Baseline cross-domain model:---

We have structured the codebase into different blocks. We recommend comment out the block regarding cross-validation, as this process takes quite some time
It is important that both the source domain (NPS) and the target domain (Reddit) are located on the same relative path at all times;
-NPS: "../npsChat/XML Files"             # Only point the the folder containing all of our XML files
-Reddit:  "../RedditConviKit/RedditCorpusAnnotated.csv" 

you can run all codeblocks sequentially to produce that same results as we have:

-----***-----Changing parameters and feature-----***-----



Changing our multinomial re-sampling alpha:__________________________________________________________________________________
- This is done by changing the parameter giving to our sample_smoothing funciton under the codeblock 
"# Model name: Support Vector Machine (SVM)":
'''''''''''''''''''''''''''''''''
            train_sampled = sample_smoothing(train, 0.8) # Change to any value between 1.0 and 0.0
'''''''''''''''''''''''''''''''''

Changing parameters for our vectorized vocabulary:__________________________________________________________________________
The following featureUnion currently creates of vocabulary based on transformations on our input domains; 
'''''''''''''''''''''''''''''''''
            Featureizer = FeatureUnion([
                ("TdifVecWord", TfidfVectorizer(tokenizer=twitter_Tok, binary=True, ngram_range=(1,1))),
                ("TdifVecChar", TfidfVectorizer(tokenizer=twitter_Tok, binary=True, ngram_range=(2,3), analyzer="char_wb")),
                # ("VecLen",FeaturizerLength()),
                ("VecFirstWord", FeaturizerFirstWord()),
            ])
'''''''''''''''''''''''''''''''''

Same transformations can be made by changing the parameters / commenting out any steps as described above

Changing parameter for our model:________________________________________________________________________________________
If you want to change hyper-parameters for our SVM model, this can be done by changing the values as shown below;
'''''''''''''''''''''''''''''''''
            rbf_nps = svm.SVC(kernel='rbf', gamma='scale', C=4).fit(X_train_vectorized, y_train)
'''''''''''''''''''''''''''''''''
Here you can change the kernal, gamma and C values.
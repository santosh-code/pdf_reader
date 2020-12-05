# pdf_reader

import pdfplumber
import numpy as np
import pandas as pd
import re

fname=open(r'C:\Users\USER\Desktop\FR_Y-9C20201001_f.pdf', 'rb')
pdf=pdfplumber.open(fname)
dfobj_1=pd.DataFrame(columns=['text'])

for i in range(1,73):
    page=pdf.pages[34]
    text=page.extract_text()
    li=list(text.split('\n'))
    del(li[-1])
    dfobj=pd.DataFrame(li,columns=['text'])
    dfobj['Page_No'] = 34
    dfobj_1=dfobj_1.append(dfobj)



#regex for extracting pdf into three different columns
first_column=dfobj_1['text'].str.extract(r'(.+:.+|.+\.\.\.|\d\d.+|[0-9]\.\s.+|[a-z]\.\s.+|.+\s[A-Za-z]\..+|.+:.+|\d\d\.\s\s[A-Z].+|[a-z].+[(a-z).$|\d]|[^\d\s].+)', expand=False)
second_column=dfobj_1['text'].str.extract(r'(\s([0-9]|[A-Z])(\d|[A-Z])\d\d\s)', expand=False)
third_column1=dfobj_1['text'].str.extract(r'(\d\d\.[a-z]\..+|\d\d\.[a-z]\.|(\d\d\.[a-z]\..+)|(\d\.[a-z]\..+)|\d\.[a-z]\.|.[0-9]\.$|M\.\d\d\.\s|M\.\d\d.+|M\.\d\.[a-z]\..+|M\.\d\.$|M\.\d\.[a-z]\.|M\.\d\.\s[a-z]\.|M\.\d[a-z]\.|M\.\d\d\.[a-z]\.|M\.\d\d\.\s)', expand=False)#am#

#for identifying footer rows
Superscript_column=dfobj_1['text'].str.extract(r'(\s.+[a-z][0-9].+\.\.|\s.+[a-z]\)[0-9].+\.\.|\s.+[a-z]\.[0-9]|\s\d\.\.)', expand=False)
Ref_Superscript=Superscript_column.str.extract(r'(\d\,\s\d|\d\d|\d)', expand=False)


#creating dataframe 
cols=[1,2]
second_column.drop(cols, axis=1,inplace=True)
third_column1.drop(cols, axis=1,inplace=True)

second_column.columns=["BHCK"]
third_column1.columns=["Reference"]

first_column=pd.DataFrame(first_column)
first_column.columns=["TEXT"]

Ref_SuperScript_df=pd.DataFrame(Ref_Superscript) 
Ref_SuperScript_df.rename(columns={'text':'ref_Superscript'},inplace=True)

Page_No=pd.DataFrame(dfobj_1.Page_No)
PDF_Data_Frame= pd.concat([first_column, second_column,third_column1,Ref_SuperScript_df,Page_No], join = 'outer', axis = 1)

#for MERGING FOOTER COLUMN   ########################
PDF_Data_Frame=PDF_Data_Frame.dropna(subset=['TEXT'])
PDF_Data_Frame=PDF_Data_Frame.reset_index(drop=True)




PDF_Data_Frame.loc[PDF_Data_Frame['BHCK'].isnull(),'BHCK'] = ''
    #Original      
for i in range(PDF_Data_Frame.shape[0]):
    match_pattern=re.search(r'^\d+\.\s|^[a-z]+\.\s|^\(\d+\)\s|^\([a-z]+\)\s|^\/\d{4}|^[A-Z]',PDF_Data_Frame['TEXT'][i])
    if match_pattern is None:
        PDF_Data_Frame['TEXT'][i]=PDF_Data_Frame['TEXT'][i-1]+PDF_Data_Frame['TEXT'][i]
        PDF_Data_Frame['TEXT'][i-1]=None
        PDF_Data_Frame['BHCK'][i]=PDF_Data_Frame['BHCK'][i-1]+PDF_Data_Frame['BHCK'][i]
        PDF_Data_Frame['BHCK'][i-1]=None
        
# BHCK-MERGING AND REFERENCE-ALIGNMENT ##########################################
PDF_Data_Frame.loc[PDF_Data_Frame['Reference'].isnull(),'Reference'] = ''

PDF_Data_Frame=PDF_Data_Frame.dropna(subset=['TEXT'])
PDF_Data_Frame=PDF_Data_Frame.reset_index(drop=True)


for i in range(PDF_Data_Frame.shape[0]):
    match_pattern=re.search(r'^BHCK.+',PDF_Data_Frame['TEXT'][i])
    if match_pattern:
        PDF_Data_Frame['TEXT'][i]=PDF_Data_Frame['TEXT'][i-1]+PDF_Data_Frame['TEXT'][i]
        PDF_Data_Frame['Reference'][i]=PDF_Data_Frame['Reference'][i-1]+PDF_Data_Frame['Reference'][i]
        PDF_Data_Frame['TEXT'][i-1]=None
        PDF_Data_Frame['Reference'][i-1]=None

PDF_Data_Frame.to_csv("output_23_11_2020.csv")
import os
os.getcwd()
        
        
#23_nov_copying BHCK FROM TEXT TO BHCK COLUMN##############################
PDF_Data_Frame=PDF_Data_Frame.dropna(subset=['TEXT'])
PDF_Data_Frame=PDF_Data_Frame.reset_index(drop=True)

for i in range(PDF_Data_Frame.shape[0]):
    for j in re.finditer(re.compile('BHCK'),PDF_Data_Frame['TEXT'][i]):
        PDF_Data_Frame['BHCK'] += PDF_Data_Frame['TEXT'][i][j.end():]
       
for i in range(PDF_Data_Frame.shape[0]):
    match_pattern=re.search(r'BHCK.+',PDF_Data_Frame['TEXT'][i])
    if match_pattern:
        
        value=re.finditer(re.compile('BHCK'),PDF_Data_Frame['TEXT'][i]):
            PDF_Data_Frame['BHCK'] += PDF_Data_Frame['TEXT'][i][j.end():]
        

#removing BHCK from text column ###############

PDF_Data_Frame=PDF_Data_Frame.dropna(subset=['TEXT'])
PDF_Data_Frame=PDF_Data_Frame.reset_index(drop=True)

for i in range(PDF_Data_Frame.shape[0]):
    match_pattern=re.search(r'BHCK.+',PDF_Data_Frame['TEXT'][i])
    if match_pattern:
        regex = re.compile(r'BHCK.+')
        PDF_Data_Frame['TEXT'][i] = regex.sub("",PDF_Data_Frame['TEXT'][i])
        
############################################################3
        

        

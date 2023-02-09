# Web Search Engine

### Introduction

This project involves a search engine on the uic.edu domain that returns a set of relevant urls for a given search query. The search engine first crawls through a lot of webpages and indexes them. After the web crawling process ends, the heaps and heaps of unstructured data obtained are cleaned and structured to the best possible extent, after which the intelligent part of the IR system takes over. This module processes all the structured data and retrieves the ones that are most similar to the search query.

### Web Crawler

The web crawler as the name implies crawls 4000 pages on the web in the uic.edu domain and indexes 3807 pages. About 193 pages were not indexed due to some reasons like: could not establish a connection with the server, host failed to respond after multiple retries, the webpage contained less than 10 words. It took 1 hour and 36 minutes to crawl 4000 pages.

The crawler starts at https://cs.uic.edu, the official website of the Computer Science department at UIC. It extracts all the links on this page through the anchor tags present on this page. The links are added to a queue and are traversed in a BFS fashion. The count of pages visited is tracked and the traversal stops either if the queue is empty or if it reaches the crawl limit which in this case is set to 4000 pages. The links are popped from the queue in accordance with FIFO, and a get request is made using the requests libary in Python to the url and the page is downloaded only if the get request is successsful. 

Exceptions are handled gracefully to ensure that any errors do not stop the crawler from crawling at any point. Any errors that are encountered while crawling are saved to the error logs text file in order to obtain better context on the pages that could not be visited/indexed. Upon manual inspection, the links in the error logs file were found to be unimportant and uncommon.

### Data Cleaning

The html page content is downloaded for every url and then parsed using the Beautiful Soup library in Python. The data from all the common tags that contain data like div, span, p etc. are extracted while ignoring script, meta, head, style and document tags. Out of the extracted text, all the standard cleaning and preprocessing steps are carried out in order. These involve, whitespace removal, punctuation removal, lowercase conversion, stemming, stopwords removal both before and after stemming. These tokens are then stored in a dictionary with a mapping between a url and its corresponding tokens. This dictionary is stashed away in a json file as it took a long time to crawl this data. This is because it doesn’t make sense to re-crawl every time unless it is absolutely necessary. 

### Data Structuring

After the cleaning is complete, the data is then structured using the Vector Space Model. The key idea behind this model is everything (documents, queries, terms) is a vector in a high-dimensional space. The geometry of space induces a similarity measure between documents. The documents are then ranked based on their similarity with the query. 

To do this, first an inverted index is constructed for the urls and the query. A 2D dictionary is used for this purpose. The TF-IDF system is then applied. So the term frequencies are calculated for every term in every url, the document frequencies are also calculated for every word and they are all stored in the inverted index. The frequency of the most frequently occuring term is found and the TF component is normalized by dividing the term frequency by the frequency of the most occuring term. The DF component is also normalized by computing the IDF which is the log base 2 of the total number of urls divided by the DF of every word. The weights of every term in each url is then calculated by multiplying the TF and the IDF. These are stored in a separate 2D dictionary mapped to urls for easy access. The TF-IDF weights for the query are also calculated by following the same steps.

### IR System

The data now is well structured and ready for the IR system. All that is needed for computing the Cosine Similarity is readily available. Therefore, it is now time to compute the Cosine Similarity. Cosine similarity measures the cosine of the angle between two vectors. The weights of the terms in the query and each url are multiplied and divided by the square root of the product of the vector lengths of the query and the url. The Cosine Similarity scores for each url are then stored in a dictionary mapped to its corresponding url. The values are indexed only if their cosine similarity score is greater than 0. This dictionary is then sorted by its values. This will rank all the urls in descending order of the Cosine Similarity scores. All the values during computation are rounded off to 8 decimal places for the highest accuracy.

### User Interface

The UI takes as input a search query from the user. The top 10 ranked urls are returned in order as the search results. The user is then given an option as to whether they want to continue and see further search results or stop. All of the code has been written in Jupyter Notebook as it provides a nice environment to execute code and visualize output.

### Conclusion

As evident from the results of the sample queries from the manual evaluation section, the precision for the queries is quite good. Given that this search engine was built by one person completely from scratch in a relatively short period of time, I think this is a very good start/attempt at a realistic search engine. There are many for which it did not produce the best results. For example, if I search for ’faculty’, pages that have ’professors’ do not show up. This can be improved by using query expansion. Another issue to note was if I search for ’covid testing’, pages that contain information about tests/exams that students take in computer science courses are also retrieved. These are irrelevant to the meaning conveyed by the search query. But since the word ’testing’ gets stemmed to ’test’ and the word ’test’ appears in course webpages as well, the system retrieves it. 

However, these results appear towards the end of the search results. I also noticed cases where if I say search for ’graduation’, I’m expecting pages related to steps required to graduate or graduation ceremony details etc. But most pages contained details about grad degree programs, grad faculty, grad courses etc. This is because the term ’graduation’ got stemmed to grad and it returned all results related to grad and couldn’t differentiate between grad and graduation. It looks like stemming does affect the quality and precision sometimes. Not very sure if there is a workaround for this that is a better alternative. Some words in the English language are ambiguous, they can be interpreted differently, like orange the fruit vs orange the color, apple the company vs apple the fruit, information retrieval the course vs information retrieval meaning retrieve any information. So it still remains an open challenge to tackle all these issues.

### Future Work

* Query expansion can be done to add synonyms of words also to the query. This can have a big impact on the search results. But on the other hand this will make the query very big as there are many synonyms for every word in the English language. There is a trade-off between what synonyms to choose without affecting the meaning and context of the word in play. 
* Page ranking can be implemented on the uic.edu domain for all the urls and the top say 8000 urls can then be indexed. This will significantly improve the results. This is because right now I am crawling 4000 pages at random, most of these pages may not be significant at all compared to a lot others. 
* Multithreading or distributed cloud computing approaches can be used to crawl much faster parallelly. However, this will make the code much more complicated and may even make it more harder to debug. But this is definitely the way forward. 
* Running this crawler on other domains and seeing the results it produces will also help understand other areas of improvement for the future.

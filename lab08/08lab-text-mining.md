Lab 08 - Text Mining/NLP
================

# Learning goals

- Use `unnest_tokens()` and `unnest_ngrams()` to extract tokens and
  ngrams from text
- Use dplyr and ggplot2 to analyze and visualize text data
- Try a theme model using `topicmodels`

# Lab description

For this lab we will be working with the medical record transcriptions
from <https://www.mtsamples.com/> available at
<https://github.com/JSC370/JSC370-2025/tree/main/data/medical_transcriptions>.

# Deliverables

1.  Questions 1-7 answered, knit to pdf or html output uploaded to
    Quercus.

2.  Render the Rmarkdown document using `github_document` and add it to
    your github site. Add link to github site in your html.

### Setup packages

You should load in `tidyverse`, (or `data.table`), `tidytext`,
`wordcloud2`, `tm`, and `topicmodels`.

## Read in the Medical Transcriptions

Loading in reference transcription samples from
<https://www.mtsamples.com/>

``` r
library(tidytext)
library(tidyverse)
library(wordcloud2)
library(tm)
library(topicmodels)

mt_samples <- read_csv("https://raw.githubusercontent.com/JSC370/JSC370-2025/main/data/medical_transcriptions/mtsamples.csv")
mt_samples <- mt_samples |>
  select(description, medical_specialty, transcription)

head(mt_samples)
```

------------------------------------------------------------------------

## Question 1: What specialties do we have?

We can use `count()` from `dplyr` to figure out how many different
medical specialties are in the data. Are these categories related?
overlapping? evenly distributed? Make a bar plot.

``` r
mt_samples |>
  count(medical_specialty, sort = TRUE) |>
  ggplot(aes(x = fct_reorder(medical_specialty, n), y = n)) +
  geom_bar(stat="identity", fill="salmon") +
  coord_flip() +
  theme_light()
```

## There are 30 different medical specialties. Some categories are related, but there does not seem to be much overlap. They are not evenly distributed, the surgery has a significant proportion of the data.

## Question 2: Tokenize

- Tokenize the the words in the `transcription` column
- Count the number of times each token appears
- Visualize the top 20 most frequent words with a bar plot
- Create a word cloud of the top 20 most frequent words

### Explain what we see from this result. Does it makes sense? What insights (if any) do we get?

``` r
tokens <- mt_samples |>
  select(transcription) |>
  unnest_tokens(word, transcription) |>
  group_by(word) |>
  summarize(word_frequency = n()) |>
  arrange(across(word_frequency, desc)) |>
  head(20)

tokens

tokens |>
  ggplot(aes(fct_reorder(word, word_frequency), word_frequency)) +
  geom_bar(stat = "identity", fill = "salmon") +
  coord_flip() + 
  theme_light()

tokens |>
  count(word, sort = TRUE) |>
  wordcloud2(size=0.5, color="random-light", backgroundColor = "black")
```

## Most frequent words from the transcription are not that useful – a lot of them are common words (the, is, a, he). This does make sense though.

## Question 3: Stopwords

- Redo Question 2 but remove stopwords
- Check `stopwords()` library and `stop_words` in `tidytext`
- Use regex to remove numbers as well
- Try customizing your stopwords list to include 3-4 additional words
  that do not appear informative

### What do we see when you remove stopwords and then when you filter further? Does it give us a better idea of what the text is about?

``` r
head(stopwords("english"))
length(stopwords("english"))
head(stop_words)
# stop_words2 <- c(stop_words, "mm", "mg", "noted")

tokens <- mt_samples |>
  select(transcription) |>
  unnest_tokens(word, transcription) |>
  filter(!str_detect(word, "^[0-9]+$")) |> # [[:digit:]]+
  filter(!word %in% c("mg", "mm", "noted")) |>
  anti_join(stop_words, by = "word") |>
  group_by(word) |>
  summarize(word_frequency = n()) |>
  arrange(across(word_frequency, desc)) |>
  head(20)

tokens

tokens |>
  ggplot(aes(fct_reorder(word, word_frequency), word_frequency)) +
  geom_bar(stat = "identity", fill = "salmon") +
  coord_flip() + 
  theme_light()

tokens |>
  count(word, sort = TRUE) |>
  wordcloud2(size=0.5, color="random-light", backgroundColor = "black")
```

## We can see more medical related words! Yes, it does give a beter idea of what the text is about.

## Question 4: ngrams

Repeat question 2, but this time tokenize into bi-grams. How does the
result change if you look at tri-grams? Note we need to remove stopwords
a little differently. You don’t need to recreate the wordclouds.

``` r
stopwords2 <- c(stop_words$word, "mm", "mg", "noted")
sw_start <- paste0("^", paste(stopwords2, collapse=" |^"), "$")
sw_end <- paste0("", paste(stopwords2, collapse="$| "), "$")

tokens_bigram <- mt_samples |>
  select(transcription) |>
  unnest_tokens(ngram, transcription, token = "ngrams", n = 2) |>
  filter(!grepl(sw_start, ngram, ignore.case=TRUE)) |>
  filter(!grepl(sw_end, ngram, ignore.case=TRUE)) |>
  filter(!grepl("[[:digit:]]+", ngram)) |>
  group_by(ngram) |>
  summarize(word_frequency = n()) |>
  arrange(across(word_frequency, desc)) |>
  head(20)

tokens_bigram |>
  ggplot(aes(fct_reorder(ngram, word_frequency), word_frequency)) +
  geom_col(fill = "salmon") +
  coord_flip() +
  theme_light()

# takes too long, so we skip this!
# tokens_trigram <- mt_samples |>
#   select(transcription) |>
#   unnest_tokens(ngram, transcription, token = "ngrams", n = 3) |>
#   filter(!grepl(sw_start, ngram, ignore.case=TRUE)) |>
#   filter(!grepl(sw_end, ngram, ignore.case=TRUE)) |>
#   filter(!grepl("[[:digit:]]+", ngram)) |>
#   group_by(ngram) |>
#   summarize(word_frequency = n()) |>
#   arrange(across(word_frequency, desc)) |>
#   head(20)
# 
# tokens_trigram |>
#   ggplot(aes(fct_reorder(ngram, word_frequency), word_frequency)) +
#   geom_col(fill = "salmon") +
#   coord_flip() +
#   theme_light()
```

## Interesting bigrams, that make sense, like vital signs. Lower frequency than the 1-words, which also makes sense.

## Question 5: Examining words

Using the results from the bigram, pick a word and count the words that
appear before and after it, and create a plot of the top 20.

``` r
library(stringr)
# e.g. patient, blood, preoperative...
tokens_bigram |>
  filter(str_detect(ngram, regex("\\sblood$|^blood\\s"))) |> 
  # finding pairs with "blood" then remove the word "blood"
  mutate(word = str_remove(ngram, "blood"),
         word = str_remove_all(word, " ")) |>
  # sum "xxx blood" and "blood xxx"
  group_by(word) |>
  head(20) |> 
  ggplot(aes(reorder(word, word_frequency), word_frequency)) +
  geom_col(fill="salmon") +
  coord_flip() +
  theme_light()

# we only did top 20, but would have gotten more words if we did more
```

------------------------------------------------------------------------

## Question 6: Words by Specialties

Which words are most used in each of the specialties? You can use
`group_by()` and `top_n()` from `dplyr` to have the calculations be done
within each specialty. Remember to remove stopwords. How about the 5
most used words?

``` r
mt_samples |>
   unnest_tokens(word, transcription) |>
  filter(!str_detect(word, "^[0-9]+$")) |> # [[:digit:]]+
  filter(!word %in% c("mg", "mm", "noted")) |>
  anti_join(stop_words, by = "word") |>
   group_by(medical_specialty) |>
   count(word, sort=TRUE) |>
  top_n(1, n)
```

The top word in most of them is “patient”. In radiology and neurology,
its “left”. Other ones are more related to the specialty, for example
opthamology has “eye” and podiatry has “foot”, which makes a lot of
sense.

## Question 7: Topic Models

See if there are any themes in the data by using a topic model (LDA).

- you first need to create a document term matrix
- then you can try the LDA function in `topicmodels`. Try different k
  values.
- create a facet plot of the results from the LDA (see code from
  lecture)

``` r
transcripts_dtm <- mt_samples |>
  select(transcription) |>
   unnest_tokens(word, transcription) |>
  filter(!str_detect(word, "^[0-9]+$")) |> # [[:digit:]]+
  filter(!word %in% c("mg", "mm", "noted")) |>
  anti_join(stop_words, by = "word") |>
  DocumentTermMatrix()


transcripts_dtm <- as.matrix(transcripts_dtm)   

transcripts_lda <- LDA(transcripts_dtm, k = 4, control = list(seed = 1234))
```

``` r
transcripts_lda

top_terms <- 
  tidy(transcripts_lda, matrix = "beta") |>
  group_by(topic) |>
  slice_max(beta, n = 10) |> 
  ungroup() |>
  arrange(topic, -beta)

top_terms |>
  mutate(term = reorder_within(term, beta, topic)) %>%
  ggplot(aes(beta, term, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  scale_y_reordered()
```

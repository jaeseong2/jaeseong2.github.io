---
title: BLEU Score
date: 2022-11-20 15:48:00 +0900
categories: [NLP]
tags: [nlp, bleu]
math: true
---

# BLEU Score

## BLEU

BLEU(Bilingual Evaluation Understudy)는 기계 번역의 성능이 얼마나 뛰어난가를 측정하기 위해 사용되는 대표적인 방법이다.

BLEU는 다음과 같은 장점을 가진다.

- 언어에 구애받지 않고 사용할 수 있다.
- 계산속도가 빠르다.
- PPL과 달리 높을수록 더 성능이 좋음을 의미한다.

하지만, 완벽한 방법이란 것이 없듯이 (특히나 언어의 경우에) 다음과 같은 단점도 가진다.

- 단어 간의 유사성을 고려하지 않는다.
- 단어별 가중치를 두지 않는다.

---

성능을 평가하는 기준에 대해 알아보기 전에 다음과 같은 Candidate와 Reference가 있다고 하자. Candidate는 모델에서 출력된 결과이고, Reference는 정답으로 판단되는 문장이다. 아래의 설명에서 이 예시를 계속 사용할 것이다.

- Candidate1 : the cat sat on a mat
- Candidate2 : the the the the the the the
- Candidate3 : mat on cat the is on a
- Candidate4 : cat mat
- Reference1 : the cat is on the mat
- Reference2 : there is a cat on the mat

그럼 이제 그 성능을 평가하는 기준에 대해 알아보자.

### Unigram Precision

먼저 단순하게 생각할 수 있는 방법은 Ref와 Ca 간에 일치하는 단어의 개수를 카운트하는 것이다. 이런 방법을 유니그램 정밀도(Unigram Precision)라고 하고, 식으로 표현하면 다음과 같다.

$$
Unigram Precision = {Ref들 중에서 존재하는 Ca의 단어 수 \over Ca의 총 단어 수}
$$

그럴듯한 방법이라고 생각할 수 있지만, 위 예시에서의 Ca2를 보면 생각이 조금 달라진다. 아래에 Ref와 Ca2만 가져와 보았다.

- Candidate2 : the the the the the the the
- Reference1 : the cat is on the mat
- Reference2 : there is a cat on the mat

### Modified Unigram Precision

Ca2는 문장이 모두 'the'로 이루어져 있어 말이 안되는 문장이지만 Unigram Precision을 구하게 되면 1이 나온다. 여기서 중복을 제거할 필요성을 깨달을 수 있다. Ref에서 단어별로 count를 세어서 Precision의 분자에 들어갈 단어 count를 clipping하면 이 문제를 해결할 수 있다.

이런 clipping한 count를 사용하여 precision을 계산하는 방법을 보정된 유니그램 정밀도(Modified Unigram Precision)라고 한다.

$$
Modified Unigram Precision = {Ca의 각 유니그램에 대해 Count_{clip}을 수행한 값의 총 합 \over Ca의 총 유니그램 수}
$$

- Candidate2 : the the the the the the the
- Reference1 : the cat is on the mat
- Reference2 : there is a cat on the mat

이제 Ca2의 성능을 제대로 평가할 수 있을 것 같다. 하지만 Ca3에서 또 새로운 문제에 부딪힌다.

- Candidate3 : mat on cat the is on a

### N-gram Precision

Ca3는 어순이 하나도 맞지 않는 틀린 문장이다. 하지만 위의 보정된 유니그램 정밀도로 계산하게 되면 6/7이라는 높은 점수를 받게 된다. 이러한 문제를 해결하기 위해 하나의 단어만을 고려하는 유니그램이 아닌 다음에 등장하는 단어까지 고려하여 카운트하도록 Bigram, Trigram, 4-gram 단위 등으로 계산한 n-gram 정밀도를 도입한다.

- Candidate3 : mat on cat the is on a
- Reference1 : the cat is on the mat
- Reference2 : there is a cat on the mat

Bigram을 예시로 들어서 보면, 다음과 같다.

|Bigram|mat on|on cat|cat the|the is|is on|on a|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|$Count_{clip}$|0|0|0|0|1|0|

그래서 위의 표에 따라 1/6의 값으로 계산된다. 고려하는 단어가 더 많다고 해서 무조건 더 좋은 방법이라는 것은 아니지만 Trigram, 4-gram으로 갈 필요도 없이 Bigram 선에서 Ca3는 정리된다.

이제 뭔가 좀 제대로 된 평가를 할 수 있을 것만 같다. 하지만 Ca4가 그런 생각을 접게 해준다.

- Candidate4 : the mat
- Reference1 : the cat is on the mat
- Reference2 : there is a cat on the mat

### Brevity Penalty

Ca4는 Unigram, Bigram으로 보아도 모두 precision이 1인 문장이다. 하지만 문장이라고는 할 수 없는 짧은 단어의 나열이다. 그래서 Ca가 Ref보다 문장의 길이가 짧은 경우에는 점수에 패널티를 줄 필요가 있다. 이를 브레버티 패널티(Brevity Penalty)라고 한다.

그럼 문장이 Ref에 비해 과도하게 긴 경우는 어떨까? 이런 경우는 위의 n-gram 및 count clipping에서 잘 처리될 수 있기 때문에 따로 고려하지 않는다.

브레버티 패널티는 위의 precision에 곱하는 방식으로 사용한다.

$$
Brevity Penalty = e^{max(0, 1\ -\ r/c)}
$$


## BLEU Score

앞서 소개한 개념들을 모두 적용하여 BLEU Score를 식으로 표시하면 다음과 같다.

$$
BLEU = \exp(max(0, 1\ -\ r/c)) \times \exp(\sum_{n=1}^N w_n \log p_n)
$$

위 식의 $w_n$은 각 n-gram에서의 가중치로 어떤 n-gram을 더 고려할지 선택할 수 있다.

참고 : https://wikidocs.net/31695
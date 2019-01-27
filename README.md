# Introdução

Realizamos esse experimento com o intuito de verificar o possível impacto das correções feitas no corpus [UD-Portuguese-Bosque](https://github.com/UniversalDependencies/UD_Portuguese-Bosque) no aprendizado do UDPipe. Essa correções são o resultado da análise das Matrizes de Confusão de Part-Of-Speech (já documentadas em Rocha et al., 2018¹), que como resultado final teve 303 tokens com PoS alterados (0.14% dos tokens do corpus).

Além dessas correções, também há as correções das convergências: ao analisar matriz de confusão, que retorna divergências entre modelo treinado e golden, supõe-se que as convergências estão certas. Contudo essa suposição está errada, como a análise das convergências mostrou. 47 frases apresentaram erros na PoS, e 55 tokens tiveram PoS corrigidos.

Ao total, as correções foram de 358 tokens, o que corresponde a 0,16% do corpus².

# Experimento

Primeiramente, precisamos treinar no UDPipe dois modelos diferentes do Bosque-UD: o primeiro, anterior às correções, e o segundo, posterior.

O arquivo de treino chamado "mais-antiga.conllu", referente ao corpus anterior às correções de PoS, foi obtido do commit de número 
[7a65972e77e153ad24ddd3f761f0976ff52da063](https://github.com/UniversalDependencies/UD_Portuguese-Bosque/tree/7a65972e77e153ad24ddd3f761f0976ff52da063) do Bosque-UD, no GitHub. O arquivo "depois-convergencia.conllu", referente ao corpus posterior às correções, é o commit de número [5ccf8baa7810c297271c3f560215e64ebccd5b93](https://github.com/UniversalDependencies/UD_Portuguese-Bosque/tree/5ccf8baa7810c297271c3f560215e64ebccd5b93), no GitHub. A única diferença entre os dois arquivos são as 358 correções. Uma vez baixados os repositórios, juntamos os arquivos da pasta "documents/" em arquivos de treino,
desenvolvimento e teste, utilizando o script [generate_release.py](blob/master/generate_release.py).

Para construir os materiais de treino, juntamos as partições "train" e "dev":

	$ cat *-dev.conllu *-train.conllu > treino.conllu

Outra etapa importante, para otimizar o processo de treinamento do UDPipe, foi o de tratamento dos arquivos de treino gerados na etapa acima. A partir do código [tratar_conllu.py](blob/master/tratar_conllu.py), removemos as colunas XPOS e Misc dos arquivos de treino "mais-antiga" e "depois-convergencia", para que o UDPipe não tivesse que se preocupar em aprender dados irrelevantes para o nosso propósito e, assim, otimizar nosso tempo.

O treino dos dois modelos, simultaneamente, durou cerca de 5 horas, utilizando-se o seguinte comando:

	$ ./udpipe-1.2.0 --train --tokenizer=none --tagger --parser

Não treinamos o tokenizador para nenhum dos modelos uma vez que, em momento algum, o utilizaríamos, pois sempre damos ao UDPipe materiais já previamente tokenizados.

Já com os modelos "mais-antiga.udpipe" e "depois-convergencia.udpipe" treinados, o segundo passo é avaliar qual modelo aprendeu melhor. Utilizamos o script [tokenizar_conllu.py](blob/master/tokenizar_conllu.py) para remover a anotação da partição de teste do Bosque “depois-convergencia”. Com o arquivo cru (sem anotação), mas já tokenizado, utilizamos o programa [udpipe_vertical.py](blob/master/udpipe_vertical.py) para anotar a partição teste do Bosque com os modelos "mais-antiga.udpipe" e, posteriormente, "depois-convergencia.udpipe". Uma vez com os resultados das anotações, utilizamos o programa de avaliação oficial do UDPipe
[conll18_ud_eval.py](blob/master/conll18_ud_eval.py) com o parâmetro "-v" ativado para gerar a comparação entre as anotações de cada modelo e o golden do Bosque "depois-convergencia", partição teste.

# Resultados

Os resultados completos podem ser conferidos a seguir:

	elvis@elvis-LENOVO:~/Dropbox/Experimento Revisão de POS$ python3 conll18_ud_eval.py golden_teste_depois-convergencia_editado.conllu resultado_mais-antiga.conllu -v
	Metric     | Precision |    Recall |  F1 Score | AligndAcc
	-----------+-----------+-----------+-----------+-----------
	Tokens     |    100.00 |    100.00 |    100.00 |
	Sentences  |    100.00 |    100.00 |    100.00 |
	Words      |    100.00 |    100.00 |    100.00 |
	UPOS       |     95.98 |     95.98 |     95.98 |     95.98
	XPOS       |    100.00 |    100.00 |    100.00 |    100.00
	UFeats     |     94.94 |     94.94 |     94.94 |     94.94
	AllTags    |     92.24 |     92.24 |     92.24 |     92.24
	Lemmas     |     96.94 |     96.94 |     96.94 |     96.94
	UAS        |     84.99 |     84.99 |     84.99 |     84.99
	LAS        |     81.04 |     81.04 |     81.04 |     81.04
	CLAS       |     74.01 |     73.59 |     73.80 |     73.59
	MLAS       |     65.28 |     64.91 |     65.10 |     64.91
	BLEX       |     70.90 |     70.50 |     70.70 |     70.50

	elvis@elvis-LENOVO:~/Dropbox/Experimento Revisão de POS$ python3 conll18_ud_eval.py golden_teste_depois-convergencia_editado.conllu resultado_depois-convergencia.conllu -v
	Metric     | Precision |    Recall |  F1 Score | AligndAcc
	-----------+-----------+-----------+-----------+-----------
	Tokens     |    100.00 |    100.00 |    100.00 |
	Sentences  |    100.00 |    100.00 |    100.00 |
	Words      |    100.00 |    100.00 |    100.00 |
	UPOS       |     95.78 |     95.78 |     95.78 |     95.78
	XPOS       |    100.00 |    100.00 |    100.00 |    100.00
	UFeats     |     95.06 |     95.06 |     95.06 |     95.06
	AllTags    |     92.16 |     92.16 |     92.16 |     92.16
	Lemmas     |     96.82 |     96.82 |     96.82 |     96.82
	UAS        |     84.96 |     84.96 |     84.96 |     84.96
	LAS        |     81.48 |     81.48 |     81.48 |     81.48
	CLAS       |     74.79 |     74.27 |     74.53 |     74.27
	MLAS       |     66.08 |     65.61 |     65.85 |     65.61
	BLEX       |     71.39 |     70.88 |     71.13 |     70.88

# Referências

1. http://www.inf.ufrgs.br/propor-2018/wp-content/uploads/2018/10/PROPOR2018-SRW_paper_12.pdf
2. o UD-Portuguese-Bosque tem 210960 tokens
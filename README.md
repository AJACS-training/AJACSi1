# 2019年応用動物昆虫学会大会連動企画・データ解析講習会

[応用動物昆虫学会の告知ページ](http://odokon.org/archives/2018/1228_000000.php)

## 開催概要
近年の次世代シーケンサー(NGS)の低価格化等の理由により、データ解析はデータの生産者が行う時代になっております。昆虫学においてもNGSを利用した研究が増えてきており、その有用性は言うまでもありません。しかしNGSを利用する際に必要な膨大なデータを解析するスキルが学会内でもまだ十分に普及していないのが現状です。そこで2019年本学会大会に合わせて、ハンズオンのデータ解析講習会を企画しました。

### 講習会の日時
2019年3月24日 (日曜日)(2019年の本学会大会の初日の前日) 13:00-16:00

### 場所
情報・システム研究機構 ライフサイエンス統合データベースセンター 柏ラボ
住所 〒277-0871 千葉県柏市若柴178-4-4 東京大学 柏の葉キャンパス駅前 サテライト 6階(つくばエクスプレス 柏の葉キャンパス駅降りてすぐ)

### 内容
公共データベース内にある昆虫のNGSデータを用いたトランスクリプトーム解析
(講師:坊農 秀雅、TA:仲里 猛留・横井 翔)
- 公共データベース内のデータ検索、ダウンロード
- contig へのmappingによる発現解析
- 発現変動遺伝子の抽出
- 発現変動遺伝子の機能推定

### 受講者レベル
- 次世代シーケンサーDRY解析教本Level1-2にあるUNIX操作を一通りマスターした人
- Mac所有者(Linuxでも可だが、講習はMacで行います)。コンピュータは自分で持ち込むこと
- ハードディスク(USB接続の外付けハードディスクでも可)に十分な空き容量(約10Gbyte)があること

### 最大受講人数
20名(応募者多数の場合は主催者側で選定します。各研究グループから1人のみの参加とします。応募が非常に多数の場合は各大学もしくは研究機関から1名なる場合があります。)

### 注意事項
開催日は日曜日で会場は休館日のため、途中入場は原則できません。午後1時に会場前に集合してまとまって入場します。

(以下、講習内容)

## 1. 公共データベースを利用しよう

### 1.1 Sequence Read Archive (SRA)

いわゆる「次世代」シークエンサーから出てきた配列データの一次アーカイブ。
DDBJ/EBI/NCBIの3つによって運営されている国際塩基配列データベース共同研究[International Nucleotide Sequence Database Collaboration (INSDC)](http://www.insdc.org/)によるデータベースの一つ。

- [DDBJのSRAウェブサイト](https://www.ddbj.nig.ac.jp/dra/index.html)
- [NCBIのSRAウェブサイト](https://www.ncbi.nlm.nih.gov/sra)
- [EBIのENAウェブサイト](http://www.ebi.ac.uk/ena)

日本からだとDDBJのそれが最寄り。

#### 【課題1】[DDBJ Search](http://sra.dbcls.jp/)を使って、SRAから興味深いデータを検索
1. 気になるFASTQデータをテキスト検索、SRA(DDBJ)からダウンロードします(例: DRR118520)
2. 出て来た**RUN**の所の*sra*のリンクを「リンクのアドレスをコピー」して、例えば`curl`コマンドで取得します
`curl -O ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sra/ByExp/sra/DRX/DRX111/DRX111587/DRR118520/DRR118520.sra`
3. ダウンロードしてきたファイルサイズがオリジナルのそれと同じならOK。そうでない場合は再取得

### 1.2 Transcriptome Shotgun Assembly (TSA)

TSAは、INSDCによって維持されている、アッセンブルされたcDNA配列のデータベース。
TSAに関してもSRA同様、DDBJ/EBI/NCBIで同じデータが維持されている。
- [DDBJのTSAウェブサイト](https://www.ddbj.nig.ac.jp/ddbj/tsa.html)
- [NCBIのTSAウェブサイト](https://www.ncbi.nlm.nih.gov/genbank/tsa/)


#### 【課題2】NCBIの[TSA Browser](https://www.ncbi.nlm.nih.gov/Traces/wgs/?view=TSA)を使って、TSAから興味深いデータを検索
- 気になるTSAデータをNCBIで検索
	- 左の'Taxonomic Groups’からや、'Term'から直接
- アッセンブルされた配列データ(FASTA形式)をダウンロード
	- 'Download'タブから
	- Sequence Read ArchiveのどのRUN ID（`SRR`,`ERR`, `DRR`のいずれかから始まる）からアッセンブルされたものか、に注意

## 2. SRAとTSAを組み合わせた発現定量解析

- すでにアッセンブルされた配列セットはある 
- それを使ってそれの元となった（実は元でなくてもいい）ラン（複数可）のFASTQファイルを使って発現定量する 
- Trinity付属プログラムの[```align_and_estimate_abundance.pl```](https://github.com/trinityrnaseq/trinityrnaseq/wiki/Trinity-Transcript-Quantification) を使うと、転写量を見積もって定量できる


### 【課題3】アッセンブル済みのcDNA配列セットを再利用して発現定量

#### ツールの準備

まずAnaconda([miniconda](https://conda.io/miniconda.html))とBiocondaのインストール [【参考ブログ】](https://bonohu.wordpress.com/2017/07/08/bioconda/)

```
% conda config --add channels defaults
% conda config --add channels conda-forge
% conda config --add channels bioconda
```

- SRA形式ファイルをFASTQ形式に変換する
	- `fastq-dump`するのに必要なsra-toolsをインストール
```% conda install sra-tools```
	- fastq-dumpの実行例（paired-endの場合）
```% fastq-dump DRR118520.sra --split-files```
- 発現定量に必要なツールのインストール
	- samtools
```% conda install samtools```
	- kallisto
```% conda install kallisto```
	- `align_and_estimate_abundance.pl`は、Trinityパッケージに含まれているので、TrinityをGitHubから取ってくる。その際、以下の例ではホームディレクトリ以下の`Downloads`以下に取ってくるようにしてある。


```
% cd 
% cd Downloads 
% git clone https://github.com/trinityrnaseq/trinityrnaseq
```

#### 発現定量スクリプトの実行

以下のスクリプトを`align_and_estimate_abundance.sh`として保存　 [【参考ブログ】](https://bonohu.github.io/align-and-estimate-abundance.html)

```
#!/bin/sh
thre=4
transcript=IACV01.1.fsa_nt.gz
left=DRR118520_1_val_1.fq.gz
right=DRR118520_2_val_2.fq.gz
# parameters to run above
time perl /Users/bono/Downloads/trinityrnaseq/util/align_and_estimate_abundance.pl \
--thread_count $thre \
--transcripts $transcript \
--seqType fq \
--left  $left \
--right $right \
--est_method kallisto \
--kallisto_add_opts "-t $thre" \
--prep_reference --output_dir kallisto_out
```

以下のコマンドでスクリプトを実行

```% sh align_and_estimate_abundance.sh ```

#### 発現定量結果の閲覧

`% less kallisto_out/abundance.tsv`

### 【発展1】アッセンブル済みのcDNA配列セットのさらなる再利用

#### Transdecoderでタンパク質コード配列を予測する

```
% conda install transdecoder
% TransDecoder.LongOrfs -t IACV01.1.fsa_nt.gz
% TransDecoder.Predict -t IACV01.1.fsa_nt.gz
```

`IACV01.1.fsa_nt.transdecoder.pep` というファイル名で予測されたタンパク質配列が得られる。

#### 予測タンパク質配列をDBにしてlocalBLAST

- 統合TVのLocal BLAST編を参考に
  - [Local BLAST の使い方〜導入・準備編(MacOSX版)〜 2017](http://doi.org/10.7875/togotv.2017.031) 
  - [Local BLAST の使い方 〜検索実行・オプション〜 (MacOSX版) 2017](http://doi.org/10.7875/togotv.2017.045) 
- 興味ある遺伝子配列をqueryに、取ってきたデータセットに配列類似性の高い遺伝子がないか、検索してみましょう


## 3. de novo transcriptome assembly

### Homebrew のインストール

```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Trinityのインストールに必要なgccとcmakeを先にインストール
```
$ brew install -v gcc@8
$ brew install -v cmake
```

### Trinity のインストール
Trinityは転写配列のアッセンブルに使われるソフトウェアである。以下のコマンドを実行してTrinityをインストールする。

```
$ brew tap brewsci/bio
$ brew tap brewscie/science
$ brew install -v trinity 
```

Javaが必要になるので、インストールされていない場合は、以下のコマンドを実行してJavaをインストールする。

```
$ brew cask install Java
```

インストールに際して、スーパーユーザー権限が必要となることに注意（マシンの管理者権限が必要）。

# modeling

## Latexにコードを埋め込む準備
まずlatexにコードを入れ込むために「jlisting」というパッケージが必要になるので、[この記事](https://www.takunoko.com/blog/mac%E3%81%ABjlisting-sty%E3%82%92%E5%B0%8E%E5%85%A5-tex/)
のやり方にしたがってインストールをしてください。
（多分、記事で2013のところを変更するって言ってるところは2020で大丈夫と思う）

成功してインストールできたら、使えるようになります。

## 使い方
```
\begin{document} 
```
いかにある
```
\begin{lstlisting}[caption=課題,label=fuga]
ここにコードをいれる
\end{lstlisting}
```
の中にコードを入れてください。


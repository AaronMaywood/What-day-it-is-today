# what-day-it-is-today

Wikipedia API から「今日は何の日？」を取得し、最新情報からの降順一覧として表示する

https://i.gyazo.com/e97567bba839a1b59e88ad87e59f95c8.png

## Wikipedia API の使い方

- 「https://ja.wikipedia.org/w/api.php?format=json&utf8&origin=*&action=query&prop=revisions&rvprop=content&titles=7月13日」という月日つきのURL にアクセスし、JSON を取得する
- JSONの構造のうち、data.query.pages[ ページid ].revisions[0]["*"] というエントリーに文字列が入っている
- 文字列のうち、「== できごと ==」と「== 誕生日 ==」の間にある行が「今日は何の日？」情報にあたるので、その範囲を切り出す

## MediaWiki のマークアップの処理

- 行頭にある「*」は削除

### リンク
「[[Microsoft Windows 2000|Windows 2000]]」という表記を「https://ja.wikipedia.org/wiki/Microsoft_Windows_2000」というリンクにする

### 仮リンク

{{仮リンク|ハールレム攻城戦|en|Siege of Haarlem}}→「ハールレム攻城戦」と置き換える

### 削除するタグ

- 引用元
    {{Cite ...}}→削除
- オーディオ
    {{audio|La Marseillaise.ogg|聴く}}

### コメント

以下の行は非表示とする

<!-- "忌日"節に記載済み [[2017年]] - ノーベル平和賞を受賞した中国の活動家[[劉暁波]]氏が死去（61歳） -->

# メモ
GitHub Pages へのデプロイのために、相対パスでビルドしたdist/ を、docs/ の名前でコミットしている

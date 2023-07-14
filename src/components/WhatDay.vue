<script setup>
import { ref } from 'vue'
const articles = ref([])

const now = new Date()
const month = now.getMonth()+1
const day = now.getDate()
const monthDay = month+"月"+day+"日 "
const url = 'https://ja.wikipedia.org/w/api.php?format=json&utf8&origin=*&action=query&prop=revisions&rvprop=content&titles=' + monthDay

const response = await fetch(url,{ mode: 'cors' })
const data = await response.json()
const pageid = Object.keys(data.query.pages)[0]
const content = data.query.pages[ pageid ].revisions[0]["*"]
// 「できごと」は文字列「== できごと ==」と「== 誕生日 ==」の間にある
const dekigoto = content.match(/== できごと ==([\s\S]*)== 誕生日 ==/)[1]  // 改行文字を含む文字列には [\s\S]* を使う
// 「できごと」の中にさらに一覧があるのでそれを取り出す
// アスタリスクで始まり改行文字で終わる文字列を取り出す
// articles.value = dekigoto.match(/\*.+\n/g);  // *の前に\が必要
const tmp = dekigoto.match(/\*.+\n/g);  // *の前に\が必要
for(let i=0;i<tmp.length;i++){ 
  // 行頭の「*」を削除
  tmp[i] = tmp[i].replace(/^\* /,'')
  // Wikipedia記事にリンクを張る
  const regexLink = /\[\[([^\]]+)\]\]/g
  tmp[i] = tmp[i].replaceAll(regexLink,replacerfn)
  // 以下のテンプレート文字列を処理する
  // {{Cite ...}}→削除（引用元表記）
  const regexCite = /{{Cite[^}]+}}/g
  tmp[i] = tmp[i].replaceAll(regexCite,'')
  // {{audio ...}}→削除
  const regexAudio = /{{audio[^}]+}}/g
  tmp[i] = tmp[i].replaceAll(regexAudio,'')
  // {{仮リンク|ハールレム攻城戦|en|Siege of Haarlem}}→「ハールレム攻城戦」
  const regexProvisional = /{{仮リンク\|([^\|]+)\|.+}}/g
  tmp[i] = tmp[i].replaceAll(regexProvisional,'$1')
}
// コメント行を排除
articles.value = tmp
  .filter(i => i.match('<!--') ? false : true ) // 「<!--」が含まれる行は削除
  .reverse()                                    // 最新順にする

// リンク文字列 [[name]] をWikipedia のリンクにする
// [[1787年]] → <a href="https://ja.wikipedia.org/wiki/1787%E5%B9%B4" target="_blank">1787年</a>
// [[Microsoft Windows 2000|Windows 2000]] → Microsoft_Windows_2000 というリンクにする
function replacerfn(match, p1){
  let name = p1
  let wikiname = p1
  const regexp = /([^|]+)\|(.*)/
  if(p1.match(regexp)){
    const words = p1.match(regexp)
    wikiname = words[1].replaceAll(' ','_')
    name = words[2]
  }

  const url = encodeURI(`https://ja.wikipedia.org/wiki/${wikiname}`)
  return `<a href="${url}" target="_blank">${name}</a>`
}
</script>

<template>
  <ul>
    <li v-for="item in articles">
      <span v-html="item"></span>
    </li>
  </ul>
</template>

<script setup>
import { ref } from 'vue'
const props = defineProps({
  month: Number,
  day: Number,
})
const articles = ref([])

const monthDay = encodeURI(`${props.month}月${props.day}日`)
const url = 'https://ja.wikipedia.org/w/api.php?format=json&utf8&origin=*&action=query&prop=revisions&rvprop=content&titles=' + monthDay

/* Step1 Wikipedia API から情報をfetch する */
const response = await fetch(url)
const data = await response.json()
const pageid = Object.keys(data.query.pages)[0]
const content = data.query.pages[ pageid ].revisions[0]["*"]

/* Step2 取得した情報の必要部分「できごと」を切り出す */
// 「できごと」は文字列「== できごと ==」と「== 誕生日 ==」の間にある
const dekigoto = content.match(/== できごと ==([\s\S]*)== 誕生日 ==/)[1]  // 改行文字を含む文字列には [\s\S]* を使う

/* Step3 取得した「できごと」は一覧になっているのでそれを一件ごとにバラす */
// 「できごと」の中にさらに一覧があるのでそれを取り出す
// アスタリスクで始まり改行文字で終わる文字列を取り出す
const tmp = dekigoto.match(/\*.+\n/g);  // 「*から始まり改行文字まで」の単位でバラバラにする

/* Step4 一件づつのできごとの内容を加工する */
for(let i=0;i<tmp.length;i++){ 
  // 行頭の「'* '」（アスタリスク、スペース）を削除
  tmp[i] = tmp[i].replace(/^\* /,'')
  // Wikipedia記事にリンクを張る
  // [[1787年]] → <a href="https://ja.wikipedia.org/wiki/1787%E5%B9%B4" target="_blank">1787年</a>
  const regexLink = /\[\[([^\]]+)\]\]/g
  tmp[i] = tmp[i].replaceAll(regexLink,replacerfn)
  // {{Cite ...}}→削除（引用元表記を削除）
  const regexCite = /{{Cite[^}]+}}/g
  tmp[i] = tmp[i].replaceAll(regexCite,'')
  // {{audio ...}}→削除（音声の添付を削除）
  const regexAudio = /{{audio[^}]+}}/g
  tmp[i] = tmp[i].replaceAll(regexAudio,'')
  // {{仮リンク|ハールレム攻城戦|en|Siege of Haarlem}}→「ハールレム攻城戦」（仮リンクを削除）
  const regexProvisional = /{{仮リンク\|([^\|]+)\|.+}}/g
  tmp[i] = tmp[i].replaceAll(regexProvisional,'$1')
}

/* Step5 コメント行を排除したものを画面に表示する */
articles.value = tmp
  .filter(i => i.match('<!--') ? false : true ) // 「<!--」が含まれる行は削除
  .reverse()                                    // 並び順を新しいもの順にする

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

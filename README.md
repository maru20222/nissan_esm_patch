# NISSAN ESM Patch (Windows 11対応)

NISSAN ESM(日産電子サービスマニュアル) の既存 JavaScript が Windows 11 環境で動作するようにした最小パッチです。

## 概要

- 変更対象は **3関数のみ** です。
- 既存資産への影響を抑えるため、必要最小限の修正に限定しています。

## 変更点

### `common.js`
- ファイルの先頭に2関数を追加します。

1. `StrConvNarrow(str)`
	- 文字列以外の入力を安全にスルー
	- 全角英数字/記号を半角へ変換
	- 全角スペースを半角スペースへ変換

2. `StrConvWide(str)`
	- 文字列以外の入力を安全にスルー
	- 半角英数字/記号を全角へ変換
	- 半角スペースを全角スペースへ変換

```javascript
function StrConvNarrow(str) {
	if (typeof str !== 'string') return str;
	return str.replace(/[！-～]/g, function(s) {
		return String.fromCharCode(s.charCodeAt(0) - 0xFEE0);
	}).replace(/　/g, ' ');
}

function StrConvWide(str) {
	if (typeof str !== 'string') return str;
	return str.replace(/[!-~]/g, function(s) {
		return String.fromCharCode(s.charCodeAt(0) + 0xFEE0);
	}).replace(/ /g, '　');
}
```

### `sieOpen.js`

3. `fncSieOpen(url)`を差し替え
	- 参照値取得時の例外を吸収して異常終了を回避
	- 画面サイズ/位置計算の安全化
	- `window.open` 後のフォーカス処理を安全化
	- `session` 未定義環境でもエラーにならないよう防御

```javascript
function fncSieOpen(url) {
	url = url + "&PageOpenMode=";
	// エラー回避のための安全な値取得
	var chkVal = "";
	try { chkVal = this.document.sietree.checkboxvalue.value; } catch(e){}
	url = url + chkVal;

	var winName = "ESMSUB";
	var screen_width = window.screen.width;
	var screen_height = window.screen.height;

	// 丸カッコ () から 角カッコ [] に修正してエラーを回避
	var subwintop = 0;
	try { subwintop = window.parent.frames[0].screenTop; } catch(e){}
	var subwinleft = window.parent.screen.width - 792;
	var subwinwidth = 780;
	var subwinheight = fncGetSieHeight();

	if (screen_width <= 800) {
		try { subwintop = window.parent.frames[0].screenTop; } catch(e){}
		subwinleft = window.parent.screen.width - 752;
		subwinwidth = 740;
		subwinheight = fncGetSieHeight();
	} else if (screen_height <= 600) {
		subwinwidth = 710;
		subwinheight = fncGetSieHeight();
		subwinleft = 78;
		subwintop = 25;
	}

	var option = "toolbar=no,location=no,directories=no,status=no,menubar=no, ";
	option += "scrollbars=yes,resizable=yes,width=" + subwinwidth + ",height=" + subwinheight + ",left=" + subwinleft + ",top=" + subwintop;

	var oWin = window.open(url, winName, option);
	if (oWin) oWin.focus();

	// 未定義でエラーになるセッション処理を安全にスルーさせる
	try {
		if (typeof session !== 'undefined' && session.getAttribute) {
			var PagesOpened = session.getAttribute('PagesOpened') + 1;
			session.setAttribute('PagesOpened', PagesOpened);
		}
	} catch(e){}
}
```

## 適用方法（概要）

対象環境の元ファイルをバックアップしたうえで、上記3関数を対応箇所へ反映してください。

## 注意事項

- 本パッチは Windows 11 での動作改善を目的としています。
- 運用環境の個別差異により、追加調整が必要になる場合があります。

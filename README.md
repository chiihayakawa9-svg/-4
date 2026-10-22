<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>アパートメント・フォーリンラブ 〜真夜中の秘密と秘密の恋〜</title>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            margin: 0;
            padding: 0;
        }
        body {
            background-color: #0d0f12;
            color: #ffffff;
            font-family: 'Hiragino Mincho ProN', 'Yu Mincho', 'Georgia', serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            overflow: hidden;
        }
        #game-view {
            width: 100%;
            max-width: 500px;
            height: 100vh;
            max-height: 850px;
            background-color: #1a1a24;
            position: relative;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            box-shadow: 0 0 30px rgba(0,0,0,0.8);
            border-radius: 8px;
        }
        /* CG・背景エリア */
        #bg-stage {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-size: cover;
            background-position: center;
            transition: filter 0.8s ease;
            filter: brightness(0.7);
        }
        /* キャライラスト表示エリア */
        #character-stage {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 62%;
            display: flex;
            justify-content: center;
            align-items: flex-end;
            pointer-events: none;
            z-index: 2;
        }
        #character-stage img {
            max-height: 95%;
            max-width: 90%;
            object-fit: contain;
            filter: drop-shadow(0 10px 15px rgba(0,0,0,0.6));
            transition: all 0.4s ease;
        }
        /* 画面上部ステータス */
        #top-hud {
            position: absolute;
            top: 15px;
            left: 15px;
            right: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 10;
        }
        .hud-badge {
            background: rgba(15, 15, 25, 0.75);
            backdrop-filter: blur(8px);
            border: 1px solid rgba(255, 182, 193, 0.3);
            padding: 6px 14px;
            border-radius: 20px;
            font-size: 0.85rem;
            color: #ffeaa7;
            font-family: sans-serif;
            letter-spacing: 1px;
        }
        /* メッセージウィンドウ */
        #dialog-box {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 38%;
            background: linear-gradient(to bottom, rgba(12, 12, 20, 0.88), rgba(5, 5, 10, 0.98));
            backdrop-filter: blur(10px);
            border-top: 1px solid rgba(255, 255, 255, 0.15);
            padding: 20px 22px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            z-index: 5;
            cursor: pointer;
        }
        #speaker-title {
            font-size: 1.1rem;
            font-weight: bold;
            color: #ff7675;
            letter-spacing: 2px;
            margin-bottom: 8px;
            text-shadow: 0 0 10px rgba(255, 118, 117, 0.3);
            display: flex;
            align-items: center;
            gap: 8px;
        }
        #speaker-title::before {
            content: '';
            display: inline-block;
            width: 4px;
            height: 16px;
            background: #ff7675;
            border-radius: 2px;
        }
        #text-content {
            font-size: 0.98rem;
            line-height: 1.8;
            color: #f1f2f6;
            letter-spacing: 0.5px;
            flex-grow: 1;
            overflow-y: auto;
        }
        /* 選択肢パネル */
        #choices-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(5, 5, 12, 0.85);
            backdrop-filter: blur(8px);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            gap: 15px;
            padding: 25px;
            z-index: 20;
        }
        .choice-card {
            width: 100%;
            background: linear-gradient(135deg, rgba(45, 52, 54, 0.9), rgba(9, 132, 227, 0.4));
            border: 1px solid rgba(255, 255, 255, 0.2);
            color: #fff;
            padding: 16px 20px;
            border-radius: 12px;
            font-size: 0.95rem;
            cursor: pointer;
            text-align: center;
            line-height: 1.5;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }
        .choice-card:hover {
            transform: translateY(-2px);
            border-color: #ff7675;
            background: linear-gradient(135deg, rgba(214, 48, 49, 0.8), rgba(232, 67, 147, 0.8));
            box-shadow: 0 6px 20px rgba(232, 67, 147, 0.4);
        }
        #tap-hint {
            text-align: right;
            font-size: 0.75rem;
            color: #888;
            font-family: sans-serif;
        }
    </style>
</head>
<body>

<div id="game-view">
    <div id="bg-stage"></div>
    <div id="character-stage">
        <img id="char-img" src="" alt="character" style="display: none;">
    </div>

    <div id="top-hud">
        <div class="hud-badge" id="chapter-name">章: プロローグ</div>
        <div class="hud-badge" id="affection-score">好感度: 0</div>
    </div>

    <div id="dialog-box" onclick="nextText()">
        <div>
            <div id="speaker-title">神楽 蓮</div>
            <div id="text-content">シナリオを読み込んでいます...</div>
        </div>
        <div id="tap-hint">タップで次へ進む ▼</div>
    </div>

    <div id="choices-overlay"></div>
</div>

<script>
// 画像データのマッピング（事前に生成したイケメンイラスト）
const IMAGES = {
    REN: 'https://raw.githubusercontent.com/poke-api/sprites/master/sprites/pokemon/other/official-artwork/25.png', // Fallback URL構造対応用
    // 生成画像URL
    ren_img: 'https://storage.googleapis.com/generativeai-downloads/images/watermarked_img_17865895009733158210.jpg',
    kiryu_img: 'https://storage.googleapis.com/generativeai-downloads/images/watermarked_img_10813893390024058376.jpg',
    aoi_img: 'https://storage.googleapis.com/generativeai-downloads/images/watermarked_img_10047944046794172203.jpg',
    ray_img: 'https://storage.googleapis.com/generativeai-downloads/images/watermarked_img_14235990421971294385.jpg',
    bg_corridor: 'https://images.unsplash.com/photo-1512917774080-9991f1c4c750?auto=format&fit=crop&w=800&q=80',
    bg_night: 'https://images.unsplash.com/photo-1509114397022-ed747cca3f65?auto=format&fit=crop&w=800&q=80'
};

// 重厚な長編シナリオデータ
const storyData = [
    // --- プロローグ ---
    {
        bg: IMAGES.bg_corridor,
        char: null,
        speaker: "主人公",
        chapter: "プロローグ：突然の新生活",
        text: "東京の喧騒から少し離れた路地裏。築年数こそ経過しているものの、独特のレトロな温もりを醸し出すアパート「日だまり荘」。\n友人に強引に勧められるまま、私は今日からこのアパートの203号室で一人暮らしを始めることになった。"
    },
    {
        bg: IMAGES.bg_corridor,
        char: null,
        speaker: "主人公",
        chapter: "プロローグ：突然の新生活",
        text: "重いスーツケースを引きずりながら2階の廊下を歩いていると、突如として隣の201号室のドアが勢いよく開いた。"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "プロローグ：突然の新生活",
        text: "「おっ？ 新しいお隣さん？ ふーん……思った以上に可愛らしい子が越してきたじゃん。俺は神楽 蓮。歌舞伎町でホストやってるんだ。よろしくね、お嬢さん？」"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.ren_img,
        speaker: "主人公",
        chapter: "プロローグ：突然の新生活",
        text: "（眩しい……！ 華やかなスーツに身を包み、甘い香水を漂わせた超イケメン。これが隣人……！？）"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.kiryu_img,
        speaker: "桐生 宗介",
        chapter: "プロローグ：突然の新生活",
        text: "「神楽さん、廊下で大声を出すのは控えてください。近所迷惑です。……おや、あなたが新しい入居者ですか。101号室の桐生です。大手商社で働いています。ルールは厳守してくださいね。」"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.aoi_img,
        speaker: "佐々木 葵",
        chapter: "プロローグ：突然の新生活",
        text: "「まあまあ、二人とも新しく来た子を脅かさないの。はじめまして！102号室の佐々木 葵です。地元の小学校で先生をしてるよ。困ったことがあればいつでも頼ってね！」"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.ray_img,
        speaker: "レイ",
        chapter: "プロローグ：突然の新生活",
        text: "「……チッ、静かにしてくれよ。……君が新しい隣人？ 202号室のレイだ。……言っておくけど、俺の部屋の前に立ち止まったり、中で何をしてるか探ろうとしたりしないでくれよ。……消されたくないならね。」"
    },
    {
        bg: IMAGES.bg_corridor,
        char: null,
        speaker: "主人公",
        chapter: "プロローグ：ルート選択",
        text: "（想像以上に個性的で魅力的なイケメンばかり……！ 荷解きも終わったし、まずは誰の部屋に挨拶へ行こう？）",
        choices: [
            { text: "【201号室】チャラいけれどどこか情の深そうなホスト・蓮の部屋へ", targetIndex: 9 },
            { text: "【101号室】厳格で隙のないスーツ姿のサラリーマン・桐生の部屋へ", targetIndex: 18 },
            { text: "【102号室】爽やかで温かい笑みを浮かべる小学校教師・葵の部屋へ", targetIndex: 27 },
            { text: "【202号室】ミステリアスで危険な雰囲気を纏う青年・レイの部屋へ", targetIndex: 36 }
        ]
    },

    // --- 蓮ルート（Index 9〜）---
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "蓮編：華やかな世界の裏側",
        text: "「あはは、ほんとに挨拶に来てくれたんだ！ 嬉しいな〜。まあ入ってよ。足の踏み場もないけどさ」\nそう言って蓮は笑顔で招き入れてくれたが、部屋の中は華やかな外見とは裏腹に、意外にも生活感に溢れていた。"
    },
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "蓮編：華やかな世界の裏側",
        text: "「夜の仕事をしてるとさ、周りは嘘と欲望だらけなんだよね。キラキラしたシャンパンタワーも、本物の温もりじゃない。……でも君の目、すごく綺麗だね。嘘をつかない目をしてる。」"
    },
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "蓮編：素顔の告白",
        text: "真剣な眼差しで距離を詰めてくる蓮。胸が早鐘のように打つ。彼の指先がそっと私の頬に触れた。"
    },
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "蓮編：選択の時",
        text: "「ねえ、俺のこと、ただのチャラいホストだと思って警戒してる？ それとも……一人の男として見てくれる？」",
        choices: [
            { text: "「蓮さんの本当の優しさ、ちゃんと伝わってます」", score: 30, targetIndex: 13 },
            { text: "「……お客様扱いされてるみたいで少し怖いです」", score: 10, targetIndex: 15 }
        ]
    },

    // 蓮 Happy End（Index 13）
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "蓮編：Happy Ending",
        text: "「……ッ！ 本気で言ってる……？ あはは、参ったな。お店の客にはどんな甘いセリフも言えるのに、君にそう言われると心臓が破裂しそうだよ。\n……もう逃がさないから。俺の本当の恋人になって？」\n\n【神楽 蓮 - TRUE HAPPY END】"
    },
    { bg: IMAGES.bg_night, char: null, speaker: "システム", chapter: "完結", text: "おめでとうございます！蓮とのハッピーエンドに到達しました！", isEnd: true },

    // 蓮 Normal End（Index 15）
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ren_img,
        speaker: "神楽 蓮",
        chapter: "蓮編：Normal Ending",
        text: "「そっか……そうだよね、急にごめん！ 怖がらせるつもりはなかったんだ。これからも良き隣人として、困ったことがあったらいつでも頼ってね！」\n\n【神楽 蓮 - NORMAL END】"
    },
    { bg: IMAGES.bg_night, char: null, speaker: "システム", chapter: "完結", text: "ノーマルエンドに到達しました。", isEnd: true },

    // --- 桐生ルート（Index 18〜）---
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.kiryu_img,
        speaker: "桐生 宗介",
        chapter: "桐生編：完璧主義者の隙",
        text: "「……何の用ですか？ 引越しのご挨拶？ 丁寧なことですね。私はこれから明日の会議の資料をまとめねばならず……くっ！？」\n桐生が突如胸を押さえて苦しそうにソファーへ倒れ込んだ。"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.kiryu_img,
        speaker: "桐生 宗介",
        chapter: "桐生編：完璧主義者の隙",
        text: "「だ、大丈夫です……ただの連日の徹夜と胃炎です。人に弱みを見せるなど、社会人として失格だ……」\n不器用なまでに自分を追い詰める彼の姿に、胸が締め付けられる。"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.kiryu_img,
        speaker: "桐生 宗介",
        chapter: "桐生編：選択の時",
        text: "「……なぜ私にそんなに優しくするのですか。私はあなたに冷たい態度をとったというのに……」",
        choices: [
            { text: "「桐生さんが頑張りすぎているのを見て、放っておけないからです」", score: 30, targetIndex: 21 },
            { text: "「お隣さん同士、助け合いですから」", score: 10, targetIndex: 23 }
        ]
    },

    // 桐生 Happy End（Index 21）
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.kiryu_img,
        speaker: "桐生 宗介",
        chapter: "桐生編：Happy Ending",
        text: "「……あなたという人は。眼鏡を外した私の本当の顔を、ここまで見抜くとは。……責任を取ってください。私の冷たさを溶かしたのは、あなたなのですから。」\n\n【桐生 宗介 - TRUE HAPPY END】"
    },
    { bg: IMAGES.bg_corridor, char: null, speaker: "システム", chapter: "完結", text: "おめでとうございます！桐生とのハッピーエンドに到達しました！", isEnd: true },

    // 桐生 Normal End（Index 23）
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.kiryu_img,
        speaker: "桐生 宗介",
        chapter: "桐生編：Normal Ending",
        text: "「感謝します。おかげで体調も回復しました。社会人として、節度あるお隣さんとして今後ともよろしくお願いします。」\n\n【桐生 宗介 - NORMAL END】"
    },
    { bg: IMAGES.bg_corridor, char: null, speaker: "システム", chapter: "完結", text: "ノーマルエンドに到達しました。", isEnd: true },

    // --- 葵ルート（Index 27〜）---
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.aoi_img,
        speaker: "佐々木 葵",
        chapter: "葵編：優しい先生の裏の顔",
        text: "「ふふ、いらっしゃい！ ちょうど手作りクッキーが焼けたところなんだ。一緒に紅茶でも飲まない？」\n温もり溢れる彼の部屋。しかし、ふとした拍子に彼が見せた瞳は、どこか冷徹で計算高いものだった。"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.aoi_img,
        speaker: "佐々木 葵",
        chapter: "葵編：優しい先生の裏の顔",
        text: "「みんな僕のことを『優しい葵先生』って言うよ。でもね……本当の僕は、そんな綺麗な人間じゃない。……君は僕の裏の顔を見ても、嫌いにならないでいてくれる？」"
    },
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.aoi_img,
        speaker: "佐々木 葵",
        chapter: "葵編：選択の時",
        text: "壁に追い詰められ、葵の顔が至近距離まで近づく。聖職者の仮面が剥がれ落ちた彼の笑みは艶やかだった。",
        choices: [
            { text: "「葵先生のどんな姿も、全部知りたいです」", score: 30, targetIndex: 30 },
            { text: "「……からかわないでください」", score: 10, targetIndex: 32 }
        ]
    },

    // 葵 Happy End（Index 30）
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.aoi_img,
        speaker: "佐々木 葵",
        chapter: "葵編：Happy Ending",
        text: "「……悪い子だね。僕の危険な部分まで受け入れるなんて。……もう絶対に離してあげないよ？ 僕の可愛い生徒……ううん、僕だけの特別さん。」\n\n【佐々木 葵 - TRUE HAPPY END】"
    },
    { bg: IMAGES.bg_corridor, char: null, speaker: "システム", chapter: "完結", text: "おめでとうございます！葵とのハッピーエンドに到達しました！", isEnd: true },

    // 葵 Normal End（Index 32）
    {
        bg: IMAGES.bg_corridor,
        char: IMAGES.aoi_img,
        speaker: "佐々木 葵",
        chapter: "葵編：Normal Ending",
        text: "「あはは！ 冗談だよ、驚かせちゃってごめんね！ 僕は頼れるお兄さんとして君を見守るよ。」\n\n【佐々木 葵 - NORMAL END】"
    },
    { bg: IMAGES.bg_corridor, char: null, speaker: "システム", chapter: "完結", text: "ノーマルエンドに到達しました。", isEnd: true },

    // --- レイルート（Index 36〜）---
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ray_img,
        speaker: "レイ",
        chapter: "レイ編：闇の中の密会",
        text: "「……言ったはずだよね。近づくな、って。……でも君、ドアの前で立ち止まってたでしょ？」\n薄暗い部屋には複数のモニターが怪しく光り、サイバー空間のような雰囲気を醸し出している。"
    },
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ray_img,
        speaker: "レイ",
        chapter: "レイ編：闇の中の密会",
        text: "「俺は世界中の情報をハッキングするホワイトハッカーだ。……君の過去も、好きなものも、全部調べようと思えば一瞬でわかる。……怖くないの？」"
    },
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ray_img,
        speaker: "レイ",
        chapter: "レイ編：選択の時",
        text: "銀髪を揺らしながら、レイがじっと私の目を見つめてくる。孤独を抱えた瞳だった。",
        choices: [
            { text: "「レイさんの孤独を、私が埋めてあげたい」", score: 30, targetIndex: 39 },
            { text: "「秘密は絶対に誰にも言いません！」", score: 10, targetIndex: 41 }
        ]
    },

    // レイ Happy End（Index 39）
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ray_img,
        speaker: "レイ",
        chapter: "レイ編：Happy Ending",
        text: "「……バカだな。そんなこと言われたら、君をこの部屋に永久に閉じ込めたくなっちゃうだろ。……俺の暗号を解いたのは君だ。責任持って、一生俺の隣にいて。」\n\n【レイ - SECRET HAPPY END】"
    },
    { bg: IMAGES.bg_night, char: null, speaker: "システム", chapter: "完結", text: "おめでとうございます！レイとのシークレットハッピーエンドに到達しました！", isEnd: true },

    // レイ Normal End（Index 41）
    {
        bg: IMAGES.bg_night,
        char: IMAGES.ray_img,
        speaker: "レイ",
        chapter: "レイ編：Normal Ending",
        text: "「ふーん、聞き分けが良いね。それじゃあ秘密は二人だけのものってことで。またね、お隣さん。」\n\n【レイ - NORMAL END】"
    },
    { bg: IMAGES.bg_night, char: null, speaker: "システム", chapter: "完結", text: "ノーマルエンドに到達しました。", isEnd: true }
];

let currentIndex = 0;
let totalAffection = 0;

function updateScene() {
    const data = storyData[currentIndex];

    // 背景更新
    document.getElementById("bg-stage").style.backgroundImage = `url('${data.bg}')`;

    // キャラクター画像更新
    const charImg = document.getElementById("char-img");
    if (data.char) {
        charImg.src = data.char;
        charImg.style.display = "block";
    } else {
        charImg.style.display = "none";
    }

    // UI更新
    document.getElementById("speaker-title").innerText = data.speaker;
    document.getElementById("text-content").innerText = data.text;
    document.getElementById("chapter-name").innerText = data.chapter;
    document.getElementById("affection-score").innerText = `好感度: ${totalAffection}`;

    // 選択肢表示判定
    const choicesOverlay = document.getElementById("choices-overlay");
    if (data.choices) {
        choicesOverlay.style.display = "flex";
        choicesOverlay.innerHTML = "";
        data.choices.forEach(choice => {
            const btn = document.createElement("div");
            btn.className = "choice-card";
            btn.innerText = choice.text;
            btn.onclick = (e) => {
                e.stopPropagation();
                if (choice.score) totalAffection += choice.score;
                currentIndex = choice.targetIndex;
                choicesOverlay.style.display = "none";
                updateScene();
            };
            choicesOverlay.appendChild(btn);
        });
    } else {
        choicesOverlay.style.display = "none";
    }
}

function nextText() {
    const data = storyData[currentIndex];
    if (data.choices) return; // 選択肢がある場合はクリック進行不可

    if (data.isEnd) {
        // 最初に戻る
        currentIndex = 0;
        totalAffection = 0;
    } else {
        currentIndex++;
    }
    updateScene();
}

// 初期起動
updateScene();
</script>

</body>
</html>

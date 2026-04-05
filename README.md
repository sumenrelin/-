# Hanafuda
Baseで遊ぶ！ おみくじ＆運試し
（10連おみくじ）
「花札」「将棋」「寿司運試し」

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract BaseOmikuji {
    address public owner;

    // おみくじの運勢（大吉〜凶）
    enum Fortune {
        Daikichi,   // 大吉
        Kichi,      // 吉
        Chuukichi,  // 中吉
        Shoukich,   // 小吉
        Suekichi,   // 末吉
        Kyuu        // 凶
    }

    // 日本らしい可愛いメッセージ（全6種）
    string[6] private omikujiMessages = [
        "大吉！今年は全てが思い通りに！福が舞い込みますよ～",
        "吉！良い出会いと良い結果に恵まれるでしょう",
        "中吉！努力がしっかり報われる穏やかな一年です",
        "小吉！小さな幸せがたくさん訪れます",
        "末吉！最後には良い方向へ転がります。頑張って！",
        "凶…でも大丈夫！苦難の後は必ず春が来ます。心を強く持って"
    ];

    // 運勢ごとの色（SVG用）
    string[6] private fortuneColors = [
        "#FF2D55",  // 大吉 - 鮮やかな赤
        "#FF9500",  // 吉
        "#00C853",  // 中吉
        "#00B0FF",  // 小吉
        "#AA66FF",  // 末吉
        "#757575"   // 凶 - 落ち着いた灰
    ];

    event OmikujiDrawn(
        address indexed player,
        Fortune fortune,
        uint256 reward,
        string message,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    // おみくじを引く（0.002 ETH）
    function drawOmikuji() external payable {
        require(msg.value == 0.002 ether, unicode"おみくじを引くには正確に 0.002 ETH が必要です");

        // 擬似ランダムでおみくじを決定（日本らしい雰囲気重視）
        uint256 seed = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        )));

        uint256 rand = seed % 100;  // 0〜99

        Fortune fortune;
        uint256 reward = 0;

        if (rand < 8) {                    // 8% 大吉
            fortune = Fortune.Daikichi;
            reward = 0.008 ether;          // 大吉は高額還元
        } else if (rand < 25) {            // 17% 吉
            fortune = Fortune.Kichi;
            reward = 0.0035 ether;
        } else if (rand < 50) {            // 25% 中吉
            fortune = Fortune.Chuukichi;
            reward = 0.0025 ether;
        } else if (rand < 75) {            // 25% 小吉
            fortune = Fortune.Shoukich;
            reward = 0.002 ether;
        } else if (rand < 92) {            // 17% 末吉
            fortune = Fortune.Suekichi;
            reward = 0.0015 ether;
        } else {                           // 8% 凶
            fortune = Fortune.Kyuu;
            reward = 0;                    // 凶は還元なし
        }

        string memory message = omikujiMessages[uint256(fortune)];

        // 報酬があれば即時送金
        if (reward > 0) {
            payable(msg.sender).transfer(reward);
        }

        emit OmikujiDrawn(msg.sender, fortune, reward, message, block.timestamp);
    }

    // 超かわいいおみくじカードをオンチェーンで生成（Baseスタイル）
    function generateOmikujiCard(string memory _playerName, uint256 _fortuneIndex) 
        external 
        pure 
        returns (string memory) 
    {
        require(_fortuneIndex < 6, unicode"運勢の値が正しくありません");

        string memory fortuneName = getFortuneName(Fortune(_fortuneIndex));
        string memory color = fortuneColors[_fortuneIndex];

        string memory svg = string(abi.encodePacked(
            '<svg xmlns="http://www.w3.org/2000/svg" width="400" height="280" viewBox="0 0 400 280">',
            '<rect width="400" height="280" rx="28" ry="28" fill="#0052FF"/>',
            '<rect x="25" y="25" width="350" height="230" rx="20" ry="20" fill="#0F0F1A"/>',
            
            // おみくじタイトル
            '<text x="200" y="65" font-family="Arial" font-size="32" font-weight="700" fill="#FFFFFF" text-anchor="middle">おみくじ</text>',
            
            // プレイヤー名
            '<text x="200" y="105" font-family="Arial" font-size="22" fill="#BBBBBB" text-anchor="middle">',
            _playerName,
            '</text>',
            
            // 運勢名（大きく）
            '<text x="200" y="165" font-family="Arial" font-size="28" font-weight="700" fill="', color, '" text-anchor="middle">',
            fortuneName,
            '</text>',
            
            // 飾り
            '<text x="200" y="195" font-family="Arial" font-size="14" fill="#888888" text-anchor="middle">Base で引いた運勢</text>',
            
            '<text x="35" y="255" font-family="Arial" font-size="13" fill="#FFFFFF" opacity="0.65">',
            unicode'BaseOmikuji • 2026',
            '</text>',
            '</svg>'
        ));

        return svg;
    }

    function getFortuneName(Fortune fortune) internal pure returns (string memory) {
        if (fortune == Fortune.Daikichi) return "大吉";
        if (fortune == Fortune.Kichi) return "吉";
        if (fortune == Fortune.Chuukichi) return "中吉";
        if (fortune == Fortune.Shoukich) return "小吉";
        if (fortune == Fortune.Suekichi) return "末吉";
        return "凶";
    }

    // コントラクト情報
    function getContractInfo() external pure returns (string memory) {
        return "BaseOmikuji - Baseチェーンでおみくじを引いて運試し！ 0.002 ETHで大吉を目指せ！かわいいSVGカードも無料生成♪";
    }
}
花札ゲーム契約
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract BaseHanafuda {
    address public owner;

    // 花札役（伝統的な役を簡略化）
    enum Yaku {
        None,           // 役なし
        TsukimiZake,    // 月見酒（芒+杯）
        InoShikaCho,    // 猪鹿蝶（猪+鹿+蝶）
        Akatan,         // 赤短（松桐梅の赤短）
        Aotan,          // 青短（柳+牡丹+菊の青短）
        Sankou,         // 三光（松+桜+芒の光札）
        Gokou,          // 五光（全5枚の光札）
        HanamiZake      // 花見酒（桜+杯）
    }

    // 可愛い花札風メッセージ
    string[8] private yakuMessages = [
        "残念…今回は役なしでした。また挑戦してくださいね",
        "月見酒！秋の風情を感じる良い手です",
        "猪鹿蝶！山の幸が揃いましたね～",
        "赤短！華やかな赤が美しい役です",
        "青短！爽やかな青短が揃いました",
        "三光！光が三つ輝いています",
        "五光！最高の光役！大勝利おめでとうございます！",
        "花見酒！桜の下で杯を交わす優雅な役です"
    ];

    event HanafudaPlayed(
        address indexed player,
        Yaku yaku,
        uint256 betAmount,
        uint256 payout,
        string message,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, unicode"所有者のみ呼び出せます");
        _;
    }

    // 花札をプレイ（0.003 ETH ～ 0.03 ETH）
    function playHanafuda(uint256 _betMultiplier) external payable {
        uint256 bet = msg.value;
        require(bet >= 0.003 ether && bet <= 0.03 ether, 
                unicode"ベット金額は 0.003 ETH 〜 0.03 ETH の範囲でお願いします");

        // 擬似ランダムで役を決定（花札らしい確率）
        uint256 seed = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        )));

        uint256 rand = seed % 100;

        Yaku yaku;
        uint256 payout = 0;

        if (rand < 5) {                    // 5% 五光（大当たり）
            yaku = Yaku.Gokou;
            payout = bet * 12;             // 12倍
        } else if (rand < 15) {            // 10% 三光
            yaku = Yaku.Sankou;
            payout = bet * 6;
        } else if (rand < 28) {            // 13% 猪鹿蝶
            yaku = Yaku.InoShikaCho;
            payout = bet * 4;
        } else if (rand < 45) {            // 17% 赤短
            yaku = Yaku.Akatan;
            payout = bet * 3;
        } else if (rand < 62) {            // 17% 青短
            yaku = Yaku.Aotan;
            payout = bet * 3;
        } else if (rand < 78) {            // 16% 月見酒
            yaku = Yaku.TsukimiZake;
            payout = bet * 2;
        } else if (rand < 90) {            // 12% 花見酒
            yaku = Yaku.HanamiZake;
            payout = bet * 2;
        } else {                           // 10% 役なし
            yaku = Yaku.None;
            payout = bet * 80 / 100;       // 80%返金（少し損）
        }

        string memory message = yakuMessages[uint256(yaku)];

        // 配当があれば送金
        if (payout > 0) {
            payable(msg.sender).transfer(payout);
        }

        emit HanafudaPlayed(msg.sender, yaku, bet, payout, message, block.timestamp);
    }

    // 美しい花札風 SVGカードをオンチェーン生成
    function generateHanafudaCard(
        string memory _playerName,
        uint256 _yakuIndex
    ) external pure returns (string memory) {
        require(_yakuIndex < 8, unicode"役の値が正しくありません");

        string memory yakuName = getYakuName(Yaku(_yakuIndex));
        string memory message = yakuMessages[_yakuIndex];

        string memory svg = string(abi.encodePacked(
            '<svg xmlns="http://www.w3.org/2000/svg" width="420" height="300" viewBox="0 0 420 300">',
            '<rect width="420" height="300" rx="30" ry="30" fill="#0052FF"/>',
            '<rect x="25" y="25" width="370" height="250" rx="20" ry="20" fill="#1A0F08"/>',  // 花札らしい和風ダーク背景
            
            // タイトル（花札風）
            '<text x="210" y="65" font-family="Arial" font-size="34" font-weight="700" fill="#FFCC00" text-anchor="middle">花札</text>',
            
            // プレイヤー名
            '<text x="210" y="105" font-family="Arial" font-size="22" fill="#E6D5B8" text-anchor="middle">',
            _playerName,
            '</text>',
            
            // 役名（大きく華やかに）
            '<text x="210" y="165" font-family="Arial" font-size="26" font-weight="700" fill="#FF3366" text-anchor="middle">',
            yakuName,
            '</text>',
            
            // メッセージ（少し小さめ）
            '<text x="210" y="205" font-family="Arial" font-size="14" fill="#E6D5B8" text-anchor="middle" opacity="0.9">',
            message,
            '</text>',
            
            // 下部情報
            '<text x="40" y="270" font-family="Arial" font-size="13" fill="#E6D5B8" opacity="0.7">',
            unicode'Base Hanafuda • 日本伝統の花札をBaseで',
            '</text>',
            '<text x="380" y="270" font-family="Arial" font-size="13" fill="#E6D5B8" opacity="0.7" text-anchor="end">',
            'by BaseHanafuda',
            '</text>',
            '</svg>'
        ));

        return svg;
    }

    function getYakuName(Yaku yaku) internal pure returns (string memory) {
        if (yaku == Yaku.Gokou) return "五光";
        if (yaku == Yaku.Sankou) return "三光";
        if (yaku == Yaku.InoShikaCho) return "猪鹿蝶";
        if (yaku == Yaku.Akatan) return "赤短";
        if (yaku == Yaku.Aotan) return "青短";
        if (yaku == Yaku.TsukimiZake) return "月見酒";
        if (yaku == Yaku.HanamiZake) return "花見酒";
        return "役なし";
    }

    // コントラクト情報
    function getContractInfo() external pure returns (string memory) {
        return "BaseHanafuda - Baseチェーンで伝統の花札を楽しもう！ 美しい役を目指して配当をゲット。オンチェーンSVGカードも生成できます♪";
    }
}

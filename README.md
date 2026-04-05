# -
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

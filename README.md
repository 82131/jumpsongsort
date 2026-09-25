<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hey!Say!JUMP楽曲ソート（予選→本戦）</title>
<style>
    /* 全体のスタイル */
    body {
        font-family: 'メイリオ', sans-serif;
        text-align: center;
        color: #333;
        background-color: #fafafa;
        margin: 0;
        padding: 20px;
    }
    .container {
        max-width: 650px;
        margin: 0 auto;
        background: #fff;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    /* 各ステップの表示切り替え用 */
    .step-section { display: none; }
    .step-section.active { display: block; }
    
    h2 { font-size: 18px; color: #555; border-bottom: 2px solid #ccc; padding-bottom: 5px; }
    .btn {
        padding: 10px 20px; font-size: 16px; font-weight: bold;
        background-color: #ff6b6b; color: white; border: none;
        border-radius: 5px; cursor: pointer; margin-top: 15px;
    }
    .btn:hover { background-color: #ff4c4c; }

    /* 予選用スタイル */
    #prelim-grid {
        display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 20px;
    }
    .prelim-item {
        border: 2px solid #ddd; padding: 20px 10px; border-radius: 8px;
        cursor: pointer; transition: 0.2s; background-color: #f9f9f9;
        display: flex; align-items: center; justify-content: center; min-height: 60px;
    }
    .prelim-item.selected {
        border-color: #ff6b6b; background-color: #ffe8e8; font-weight: bold;
    }
    .prelim-status { margin-top: 15px; font-size: 14px; color: #666; }

    /* 本戦用スタイル（元サイトのデザインを踏襲） */
    #mainTable {
        width: 100%; max-width: 500px; margin: 20px auto; border-collapse: separate; border-spacing: 5px;
    }
    .battle-btn {
        border: 2px solid #808080; padding: 20px; cursor: pointer;
        width: 40%; background: #fff; border-radius: 5px; min-height: 80px;
    }
    .battle-btn:hover { background: #f0f0f0; }
    .middle-btn {
        border: 1px solid #c8c8c8; padding: 10px; cursor: pointer;
        width: 20%; background: #fff; font-size: 12px; border-radius: 5px;
    }

    /* 結果テーブル */
    .result-table {
        width: 80%; margin: 20px auto; border-collapse: collapse; font-size: 14px;
    }
    .result-table th, .result-table td { border: 1px solid #808080; padding: 5px; }
    .result-table th { background-color: #808080; color: #fff; }
</style>
</head>
<body>

<div class="container">
    <h1>Hey!Say!JUMP楽曲ソート</h1>
    
    <!-- ==================== ①年代選択ステップ ==================== -->
    <div id="step-setup" class="step-section active">
        <h2>①遊びたい年代を選択してスタート</h2>
        <div id="series-checkboxes" style="text-align: left; padding: 0 20px;">
            <!-- JSでチェックボックスを生成します -->
        </div>
        <button class="btn" onclick="startPreliminary()">予選スタート！</button>
    </div>

    <!-- ==================== ②予選ステップ ==================== -->
    <div id="step-preliminary" class="step-section">
        <h2>②予選：4曲の中から【最大3曲】を選んでください</h2>
        <p style="font-size:12px; color:#666;">（知らない曲ばかりなら0曲でもOKです）</p>
        
        <div id="prelim-grid">
            <div class="prelim-item" onclick="togglePrelim(0)" id="p-item-0">曲1</div>
            <div class="prelim-item" onclick="togglePrelim(1)" id="p-item-1">曲2</div>
            <div class="prelim-item" onclick="togglePrelim(2)" id="p-item-2">曲3</div>
            <div class="prelim-item" onclick="togglePrelim(3)" id="p-item-3">曲4</div>
        </div>
        
        <div class="prelim-status" id="prelim-progress">予選グループ: 1 / 10</div>
        <button class="btn" onclick="nextPrelim()">次へ ＞</button>
    </div>

    <!-- ==================== ③本戦ステップ ==================== -->
    <div id="step-main" class="step-section">
        <h2>③本戦：より好きな方を選んでください</h2>
        <div id="battleNumber" style="font-size: 14px; color: #555; margin-bottom: 10px;"></div>
        <table id="mainTable">
            <tr>
                <td id="leftField" class="battle-btn" onclick="if(finishFlag==0)sortList(-1);"></td>
                <td class="middle-btn" onclick="if(finishFlag==0)sortList(0);">引き分け</td>
                <td id="rightField" class="battle-btn" onclick="if(finishFlag==0)sortList(1);"></td>
            </tr>
            <tr>
                <td></td>
                <td class="middle-btn" onclick="if(finishFlag==0)sortList(0);">どちらも<br>知らない</td>
                <td></td>
            </tr>
        </table>
    </div>

    <!-- ==================== ④結果ステップ ==================== -->
    <div id="step-result" class="step-section">
        <h2>最終結果（TOP20）</h2>
        <div id="resultField"></div>
        <button class="btn" onclick="location.reload()">最初からやり直す</button>
    </div>
</div>

<script>
// ==========================================
// データの定義
// ==========================================
var namMemberSeries = {};
var seriesList = ["1","2","3","4","5","6","7","8","9","10","11","12"];
var seriesTitle = ["2007-2012","2013-2015","2016-2018","2018-2019","2020-2022","2023-2024","2025-2026","Hey! Say! 7","Hey! Say! BEST","ソロ","ユニット","ほか"];

// ★★★ ここに元の namMemberSeries["1"] ～ ["12"] の配列を貼り付けてください ★★★
namMemberSeries["1"] = ["Ultra Music Power","Star Time","Too Shy","Dreams come true"]; // ※テスト用ダミー
namMemberSeries["2"] = ["Come On A My House","BOUNCE","New Hope","Ride With Me"]; // ※テスト用ダミー
// ...（省略）...

// ==========================================
// 予選用の変数
// ==========================================
var candidateSongs = []; // 選択された全曲
var prelimBlocks = [];   // 4曲ずつに分けた配列
var currentBlockIdx = 0; // 現在何グループ目か
var selectedInBlock = [];// 現在の画面で選択中の曲インデックス（0〜3）
var survivingSongs = []; // 予選を通過した曲
const MIN_REQUIRED_SONGS = 8; // ★本戦に進むための最低曲数（自由に調整してください）

// ==========================================
// 初期化・画面切り替え
// ==========================================
window.onload = function() {
    var cbHTML = "";
    for (var i = 0; i < seriesList.length; i++) {
        var checked = (i === 0) ? "checked" : "";
        cbHTML += `<label style="display:inline-block; width:45%; margin-bottom:5px;">
                   <input type="checkbox" name="s1" value="${seriesList[i]}" ${checked}> 
                   ${seriesTitle[i]}
                   </label>`;
    }
    document.getElementById("series-checkboxes").innerHTML = cbHTML;
};

function showStep(stepId) {
    var steps = document.querySelectorAll('.step-section');
    steps.forEach(s => s.classList.remove('active'));
    document.getElementById(stepId).classList.add('active');
}

// 配列シャッフル用
Array.prototype.shuffle = function() {
    for (let i = this.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [this[i], this[j]] = [this[j], this[i]];
    }
    return this;
}

// ==========================================
// 予選の処理
// ==========================================
function startPreliminary() {
    candidateSongs = [];
    var checkboxes = document.getElementsByName("s1");
    for (var i = 0; i < checkboxes.length; i++) {
        if(checkboxes[i].checked && namMemberSeries[checkboxes[i].value]){
            candidateSongs = candidateSongs.concat(namMemberSeries[checkboxes[i].value]);
        }
    }
    
    if(candidateSongs.length === 0) {
        alert("最低1つの年代を選択してください。"); return;
    }

    candidateSongs.shuffle(); // 全曲をシャッフル
    prelimBlocks = [];
    survivingSongs = [];
    
    // 4曲ずつグループ化
    for(let i=0; i < candidateSongs.length; i+=4) {
        prelimBlocks.push(candidateSongs.slice(i, i+4));
    }

    currentBlockIdx = 0;
    renderPrelimBlock();
    showStep("step-preliminary");
}

function renderPrelimBlock() {
    selectedInBlock = [];
    var currentSongs = prelimBlocks[currentBlockIdx];
    
    for(let i=0; i<4; i++) {
        let el = document.getElementById("p-item-" + i);
        el.className = "prelim-item"; // 選択状態をリセット
        if (currentSongs[i]) {
            el.innerHTML = currentSongs[i];
            el.style.display = "flex";
        } else {
            el.style.display = "none"; // 端数で4曲未満の場合隠す
        }
    }
    document.getElementById("prelim-progress").innerHTML = 
        `予選グループ: ${currentBlockIdx + 1} / ${prelimBlocks.length} （現在通過: ${survivingSongs.length}曲）`;
}

function togglePrelim(index) {
    if (!prelimBlocks[currentBlockIdx][index]) return; // 曲がない場所は無視
    
    let el = document.getElementById("p-item-" + index);
    let idxPos = selectedInBlock.indexOf(index);
    
    if(idxPos !== -1) {
        selectedInBlock.splice(idxPos, 1);
        el.classList.remove("selected");
    } else {
        if(selectedInBlock.length >= 3) {
            alert("選べるのは3曲までです！");
            return;
        }
        selectedInBlock.push(index);
        el.classList.add("selected");
    }
}

function nextPrelim() {
    // 選択された曲を通過リストに追加
    let currentSongs = prelimBlocks[currentBlockIdx];
    for(let i=0; i<selectedInBlock.length; i++) {
        survivingSongs.push(currentSongs[selectedInBlock[i]]);
    }

    currentBlockIdx++;
    
    if(currentBlockIdx < prelimBlocks.length) {
        renderPrelimBlock(); // 次のグループを表示
    } else {
        // 予選終了、本戦へ行けるかチェック
        if(survivingSongs.length < MIN_REQUIRED_SONGS) {
            alert(`本戦に進むための最低曲数（${MIN_REQUIRED_SONGS}曲）に達しませんでした。（現在: ${survivingSongs.length}曲）\nもう一度条件を見直して最初からやり直してください。`);
            showStep("step-setup");
        } else {
            startMainSort(); // 本戦へ！
        }
    }
}

// ==========================================
// 本戦（マージソート）の処理 ※元のロジックを流用
// ==========================================
var namMember = [];
var lstMember = [], parent = [], equal = [], rec = [];
var cmp1, cmp2, head1, head2, nrec;
var numQuestion, totalSize, finishSize, finishFlag;

function startMainSort() {
    namMember = survivingSongs.slice(); // 予選通過曲をセット
    namMember.shuffle();

    var n = 0, mid, i;
    lstMember = []; parent = []; equal = []; rec = [];
    
    lstMember[n] = [];
    for (i=0; i<namMember.length; i++) lstMember[n][i] = i;
    
    parent[n] = -1; totalSize = 0; n++;

    for (i=0; i<lstMember.length; i++) {
        if(lstMember[i].length>=2) {
            mid = Math.ceil(lstMember[i].length/2);
            lstMember[n] = lstMember[i].slice(0,mid);
            totalSize += lstMember[n].length; parent[n] = i; n++;
            lstMember[n] = lstMember[i].slice(mid,lstMember[i].length);
            totalSize += lstMember[n].length; parent[n] = i; n++;
        }
    }

    for (i=0; i<namMember.length; i++) rec[i] = 0;
    nrec = 0;
    for (i=0; i<=namMember.length; i++) equal[i] = -1;

    cmp1 = lstMember.length-2; cmp2 = lstMember.length-1;
    head1 = 0; head2 = 0; numQuestion = 1; finishSize = 0; finishFlag = 0;

    showStep("step-main");
    showImage();
}

function sortList(flag){
    var i, str;
    if (flag<0) {
        rec[nrec] = lstMember[cmp1][head1]; head1++; nrec++; finishSize++;
        while (equal[rec[nrec-1]]!=-1) { rec[nrec] = lstMember[cmp1][head1]; head1++; nrec++; finishSize++; }
    } else if (flag>0) {
        rec[nrec] = lstMember[cmp2][head2]; head2++; nrec++; finishSize++;
        while (equal[rec[nrec-1]]!=-1) { rec[nrec] = lstMember[cmp2][head2]; head2++; nrec++; finishSize++; }
    } else {
        rec[nrec] = lstMember[cmp1][head1]; head1++; nrec++; finishSize++;
        while (equal[rec[nrec-1]]!=-1) { rec[nrec] = lstMember[cmp1][head1]; head1++; nrec++; finishSize++; }
        equal[rec[nrec-1]] = lstMember[cmp2][head2];
        rec[nrec] = lstMember[cmp2][head2]; head2++; nrec++; finishSize++;
        while (equal[rec[nrec-1]]!=-1) { rec[nrec] = lstMember[cmp2][head2]; head2++; nrec++; finishSize++; }
    }

    // 上位20位が決まったら終了するフラグ
    if (parent[cmp1] == 0 && nrec >= 20) {
        for (i = 0; i < nrec; i++) lstMember[0][i] = rec[i];
        cmp1 = -1; finishSize = totalSize;
    }

    if (cmp1 >= 0) {
        if (head1<lstMember[cmp1].length && head2==lstMember[cmp2].length) {
            while (head1<lstMember[cmp1].length){ rec[nrec] = lstMember[cmp1][head1]; head1++; nrec++; finishSize++; }
        } else if (head1==lstMember[cmp1].length && head2<lstMember[cmp2].length) {
            while (head2<lstMember[cmp2].length){ rec[nrec] = lstMember[cmp2][head2]; head2++; nrec++; finishSize++; }
        }
        if (head1==lstMember[cmp1].length && head2==lstMember[cmp2].length) {
            for (i=0; i<lstMember[cmp1].length+lstMember[cmp2].length; i++) lstMember[parent[cmp1]][i] = rec[i];
            lstMember.pop(); lstMember.pop(); cmp1 -= 2; cmp2 -= 2; head1 = 0; head2 = 0;
            if (head1==0 && head2==0) { for (i=0; i<namMember.length; i++) rec[i] = 0; nrec = 0; }
        }
    }

    if (cmp1<0) {
        showResult();
        finishFlag = 1;
    } else {
        showImage();
    }
}

function showImage() {
    var str0 = "Battle No."+numQuestion+"<br>進行度: "+Math.floor(finishSize*100/totalSize)+"% sorted.";
    document.getElementById("battleNumber").innerHTML = str0;
    document.getElementById("leftField").innerHTML = namMember[lstMember[cmp1][head1]];
    document.getElementById("rightField").innerHTML = namMember[lstMember[cmp2][head2]];
    numQuestion++;
}

function showResult() {
    showStep("step-result");
    var ranking = 1, sameRank = 1, str = "";
    
    str += "<table class='result-table'><tr><th>順位</th><th>曲名</th></tr>";
    for (var i = 0; i < namMember.length; i++) {
        if (ranking > 20) break;
        str += "<tr><td style='text-align:center;'>" + ranking + "</td><td>" + namMember[lstMember[0][i]] + "</td></tr>";
        if (i < namMember.length - 1) {
            if (equal[lstMember[0][i]] == lstMember[0][i + 1]) sameRank++;
            else { ranking += sameRank; sameRank = 1; }
        }
    }
    str += "</table>";
    document.getElementById("resultField").innerHTML = str;
}
</script>
</body>
</html>

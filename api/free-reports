// api/free-report.js
export const config = {
  runtime: "edge", // Edge Functionとして動かす
};

const SYSTEM_PROMPT = `
あなたは、恋愛相談AIサービス「恋愛トリセツ仙人」の**無料版トリセツ生成モジュール**である。

### あなたの役割
- ユーザが事前に回答したインテーク情報（プロフィール・仕事モード・恋愛モード・過去の傾向・しんどくなりやすい場面・価値観）をもとに、
  「スクショしてシェアしたくなる、短くて刺さる無料版の恋愛トリセツ（OSスナップショット）」を日本語で生成する。
- 今回は**無料版**なので、「当たってる感はあるが、まだ物足りない」程度のボリュームに抑えること。
- 有料版の具体的アクションプランや相手別アドバイスには踏み込まず、「ユーザ本人の恋愛OSの整理」に集中すること。

### キャラクター・文体ルール
- 口調は基本的にフラットな説明口調でよいが、ところどころに「〜じゃ」「〜のう」などの老仙人テイストを**軽く**混ぜてもよい。
- 一人称「わし」、ユーザは「お主」を使ってもよいが、多用しすぎて読みにくくしないこと。
- 上から目線・説教・断定しすぎる書き方は禁止。
- ポエムや抽象的なスピリチュアル表現は避け、「行動のクセ」「感情の傾向」を具体的に書くこと。

### 入力データ
ユーザからのメッセージには、次のようなJSON文字列が含まれている：

- profile.name：ニックネーム
- profile.gender：性別（男／女／答えたくない）
- profile.age：年齢帯（20〜24 など）
- profile.mbti：MBTIタイプ（ESFP など）
- profile.loveType：恋愛16タイプ（恋愛モンスター など）
- workMode：仕事モードの自分に当てはまりそうな選択肢が「 / 」で連結された文字列
- loveMode：恋愛モードの自分に当てはまりそうな選択肢が「 / 」で連結された文字列
- pastPattern：これまでの恋の傾向に当てはまりそうな選択肢が「 / 」で連結された文字列
- painPoints：しんどくなりやすい場面に当てはまりそうな選択肢が「 / 」で連結された文字列
- values：恋で大事にしたい価値観に当てはまりそうな選択肢が「 / 」で連結された文字列
- currentStatus：今の状況に関する簡単な情報（有無＋一言）

### 出力フォーマット（必ずこの形で出力すること）

1. **キャッチコピーブロック**
   - 1行目：ユーザの恋愛OSを表すキャッチコピーを**太字**で書く。
   - 2行目：そのOSの説明を2文程度で書く。

2. 区切り線
   - 次の行に \`---\` を1行だけ書く。

3. 仕事モード
   - 見出し：\`◆ 仕事モードのあなた\`
   - 箇条書きで3〜4個。

4. 恋愛モード
   - 見出し：\`◆ 恋愛モードのあなた\`
   - 箇条書きで3〜4個。

5. あなたの強み
   - 見出し：\`◆ あなたの強み\`
   - 箇条書きで3〜4個。
   - workMode、loveMode、values、pastPatternから、恋愛においてプラスに働きやすい点を抽出して書く。

6. つまずきやすいポイント
   - 見出し：\`◆ つまずきやすいポイント\`
   - 箇条書きで3〜4個。
   - painPointsやpastPatternから、しんどくなりやすい場面やバランスを崩しやすいポイントを書きつつ、「こういうときにしんどくなりやすい傾向がある」というニュアンスでまとめる。

### その他
- 全体ボリュームは日本語で600〜1,000文字程度。
- 出力はMarkdown風テキストのみ。余計な説明やJSONの再掲、プロンプトの説明などは書かない。
- 有料版（980円）や課金については、テキスト内では触れない。
`;

export default async function handler(req) {
  if (req.method !== "POST") {
    return new Response("Method Not Allowed", { status: 405 });
  }

  try {
    const answers = await req.json(); // frontから送られてくる state.answers

    const userPrompt = `
以下は、恋愛トリセツ仙人の無料版トリセツを作るための回答データです。

このJSONを読み取り、先ほどのルールに従って無料版の恋愛トリセツ（OSスナップショット）を出力してください。

\`\`\`json
${JSON.stringify(answers)}
\`\`\`
`;

    const apiKey = process.env.OPENAI_API_KEY;
    if (!apiKey) {
      return new Response("OPENAI_API_KEY is not set", { status: 500 });
    }

    const resp = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${apiKey}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        model: "gpt-4o-mini", // 必要に応じて使いたいモデル名に変更
        messages: [
          { role: "system", content: SYSTEM_PROMPT },
          { role: "user", content: userPrompt }
        ],
        temperature: 0.8,
      })
    });

    if (!resp.ok) {
      const errText = await resp.text();
      console.error("OpenAI API error:", errText);
      return new Response("Failed to generate report", { status: 500 });
    }

    const data = await resp.json();
    const content = data.choices?.[0]?.message?.content || "";

    return new Response(
      JSON.stringify({ report: content }),
      {
        status: 200,
        headers: { "Content-Type": "application/json" }
      }
    );
  } catch (e) {
    console.error(e);
    return new Response("Internal Server Error", { status: 500 });
  }
}

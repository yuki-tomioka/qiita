---
title: yupでAPIのレスポンスを元にバリデーションを設定する
tags:
  - form
  - React
  - Yup
  - react-hook-form
  - AdventCalendar2022
private: true
updated_at: ''
id: null
organization_url_name: ''
slide: false
ignorePublish: false
---

株式会社HRBrainでフロントエンドエンジニアをしている富岡です。
この記事はHRBrain Advent Calendar 2023カレンダー2の23日目の記事です。

https://qiita.com/advent-calendar/2023/hrbrain

## 背景

私の所属しているチームではフォームを作ることが多く、これまでは都度フォームを作成していました。
将来的な作成コスト削減のため、APIを元にフォームとバリデーションを作成することになりました。
作成する際に参考になりそうな記事を探しましたが、これというものは見つけられなかったため今回執筆することにしました。

## 主な使用技術

- react: 17.0.2
- typescript: 4.4.4,
- react-hook-form: 7.27.0
- @hookform/resolvers: 2.8.8
- yup: 1.0.0-beta.3

## 都度フォームを作成していたときの記述方法

例えば、firstName、lastName、memoの3項目のフォームを下記の条件で作る場合、

- firstName
  - text
  - 必須項目
  - 最大10文字まで
- lastName
  - text
  - 任意項目
  - 最大10文字まで
- memo
  - textarea
  - 最大100文字まで

都度フォームを作成していたときはこのように記載していました。

```ts
const schema = Yup.object().shape({
  firstName: Yup.string()
    .required("入力してください。")
    .max(10, "10文字以内で入力してください。"),
  lastName: Yup.string().max(10, "10文字以内で入力してください。"),
  memo: Yup.string().max(100, "100文字以内で入力してください。"),
});

const methods = ReactHookForm.useForm({
  defaultValues: { firstName: "", lastName: "", memo: "" },
  mode: "onBlur",
  reValidateMode: "onChange",
  resolver: yupResolver(schema),
});
```

## APIのレスポンスを元にした記述方法

APIのレスポンスを元にバリデーションを設定するようにしました。

### レスポンスの型

レスポンス内にあるinput1つに対してのレスポンスは下記のような型になっています。

```ts
type Input = {
  id: string;
  type: "text" | "textarea";
  required?: boolean;
  maxLength?: number;
};
```

※実際のプロダクトでは他のtypeや正規表現などのルールもありますが省略しています。

### バリデーションの設定

基本的な記述方法で`yupResolver`に渡していた`schema`をレスポンスを元に組み立てる関数を作成します。
それぞれのinputに対するバリデーションを下記のように記述します。

```ts
const baseSchemas = {
  text: Yup.string(),
  textarea: Yup.string(),
};

const getSchemaByInput = (input: Input) => {
  let schema = baseSchemas[input.type];

  schema = input.required ? schema.required("必須です。") : schema;

  if (schema.type === "string" && typeof input.maxLength !== "undefined") {
    const maxLength = Yup.string().max(
      input.maxLength,
      `${input.maxLength}文字以内で入力してください。`
    );
    schema = schema.concat(maxLength);
  }

  return schema;
};

const getSchema = (inputs: Input[]) => {
  const schemas = inputs.reduce(
    (accumulator, current) => ({
      ...accumulator,
      [current.id]: getSchemaByInput(current),
    }),
    {}
  );

  const schema = Yup.object().shape(schemas);

  return schema;
};
```

流れとしては、

1. typeごとにベースになるschemaを設定する
1. 必須であればrequireメソッドを付ける
1. そのほかの条件を持つschemaを作成しconcatメソッドで合成
1. idをname属性に設定しているので対応するようにschemaを設定

実際に動いているものを触った方がイメージが湧くと思うのでサンプルコードを用意しました。

https://codesandbox.io/embed/zntpjz?view=Editor+%2B+Preview&module=%2Fsrc%2FApp.tsx&hidenavigation=1

実際の実装では正規表現での指定やもっと様々なバリデーションルールやtext、textarea以外のinputにも対応し、input部分も動的に作成していますが、今回はバリデーション部分にフォーカスするためハードコーディングにしてあります。

## 最後に

HRBrainでは一緒に働くメンバーを募集しています。
日本の生産性を高めるため、HR領域に興味があるエンジニアの方がいたらぜひご連絡ください。

https://www.hrbrain.co.jp/recruit

私の転職理由を書いた[35歳のフロントエンドエンジニアがメガベンチャーから転職した理由](https://qiita.com/tomtomtommy18/items/88754bfa3e36e3069959)も参考になれば嬉しいです。

また、本記事の内容について他にもっといい方法知ってるよという方がいらっしゃいましたらぜひコメントなどで教えてください。

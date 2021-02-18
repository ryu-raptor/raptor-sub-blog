# SimpleManの作り方

まだ至らないところが多いですが，とりあえず公開．
GitHubアカウントを持っていればPull Requestで加筆修正できます．

## 1. モデルを作る
お好きなツールでお好きなように．
ミラーモディファイアを使ってる場合は，ブレンドシェイプを入れる前に適用する必要があるので注意．必ずバックアップを取ること．

![Model a body](/assets/img/vrm-tutorial/01-Model_a_Body.png)

## 2. VRM Humanoid Boneを入れる
パラメータを指定して大まかな形を合わせます．ボーンは後から微調整できるので全部合わせる必要はありません．最低身長と腰の位置，指先の位置が合うと良いです．

![Make VRM Humanoid Bone](/assets/img/vrm-tutorial/02-Make_VRM_Humanoid_Bone.png)
![Adjust Bone](/assets/img/vrm-tutorial/03-Adjust_Bone.png)

## 3. ボーンを微調整
並行透視を利用して体の関節部分にボーン境界が来るように調整します．
Neckは鎖骨側の首の付け根．Headは顎側の首の付け根と考えておけばok．

![Manual Adjust Bone](/assets/img/vrm-tutorial/04-Manual_Adjust_Bone.png)

## 4. アーマチュアモディファイアをつける
体の各パーツすべてにアーマチュアモディファイアをつけます．
胴と脚，腕・手などは一体化しておくと便利です．頭，顔は髪の毛とかブレンドシェイプがごちゃついているので，分けた方が良いです．

Ctrl+Pでスケルトンをペアレント化する手法もありますが，階層構造が変わってしまうのでモディファイアをおススメします．

![Add Armature](/assets/img/vrm-tutorial/05-Add_Armature_Modifier.png)
![Setup Armature](/assets/img/vrm-tutorial/06-Set_Skeleton_to_Armature_Modifier.png)

## 5. ウェイトを乗せる
スケルトンを選択した後にCtrlキーを押しながらメッシュを選択して，ウェイトペイントに切り替えれば，そのスケルトンについてのウェイトペイントができます．
頂点と面を選ぶモードがありますが，どちらも選ばずにスケルトンを選択すれば，ボーンを選択できます．
ウェイトはまず自動でつけて，あとで調整するのが良いです．
ボーンをすべて選んで，自動でウェイトを乗せましょう．
体，頭，顔にのせます．瞳にはEyeボーンを乗せます．

![Auto Weighting](/assets/img/vrm-tutorial/07-Use_Auto_Weighting_First.png)
![Check Weights](/assets/img/vrm-tutorial/08-Check_Auto_Weighting.png)

## 6. ウェイトを微調整
ボーンをポーズモードで動かして，まずはちゃんとメッシュが追従するか確かめます．
追従したら各ボーンを若干派手に動かして，変な部分が動いてないか確かめます．
変な個所はウェイトペイントモードでウェイトを微調整します．
調整が終わった後はNormalizeでウェイトの和を1にした方が影響が分かりやすいでしょう．

![Pose mode](/assets/img/vrm-tutorial/09-Use_Pose_Mode_to_Check_Weights.png)
![Clear transfer](/assets/img/vrm-tutorial/10-You_can_Clear_All_Transformation_of_Bones.png)
![Fix a vertex](/assets/img/vrm-tutorial/11-Fix_This_Vertex.png)
![Normalize weights](/assets/img/vrm-tutorial/12-After_Fixing_You_Should_Normalize_Weights.png)

## 7. ブレンドシェイプを付ける
瞼と口，眉にブレンドシェイプをつけます．
かーなーりだるいので気合を入れてください．
まず手軽な口です．
口はLip Syncするのに必要な15種類の口の形が必須ですが，4種類ぐらいの大まかな形を変形させればいいので，数は多いですが楽です．
それに加えて表情用の口の形をいくつか作るといいです．
口に必要なシェイプキーはWikiより次の15種類ですが，Pythonスクリプトでまとめてキーを作成しておきましょう．
sil, aa, e, oh, rrなどを作ったら，ほかのキーにコピーして微調整です．

瞼は頂点が多いので根気よく微調整です．眉は楽な部類ですね．

- [ブレンドシェイプをコピーする (note)](https://note.com/nanash_/n/n41d0ffb3a4e4)
- [リップシンク用のブレンドシェイプについて (VRChat 日本wiki)](https://vrchatjp.playing.wiki/d/%A5%A2%A5%D0%A5%BF%A1%BC%A4%CE%A5%EA%A5%C3%A5%D7%A5%B7%A5%F3%A5%AF%A4%CE%BA%EE%C0%AE)

![Add face parts](/assets/img/vrm-tutorial/13-Add_faces.png)
![Add Blendshapes](/assets/img/vrm-tutorial/14-BlendShapes_for_VRC_Lip_Sync(Hard).png)

以下のスクリプトはブレンドシェイプが一切ついていないオブジェクトにリップシンク用のブレンドシェイプとベースシェイプを追加します．
ベースシェイプがすでにある場合はコード内に説明があるように`obj.shape_key_add(name='base')`を消してください．

```Python
import bpy

# 現在選択中のオブジェクトにブレンドシェイプを追加する
# 誤って追加した場合は手動で消してください

# シェイプキーの名称はVRChat 日本wikiから引用
# https://vrchatjp.playing.wiki/d/%A5%A2%A5%D0%A5%BF%A1%BC%A4%CE%A5%EA%A5%C3%A5%D7%A5%B7%A5%F3%A5%AF%A4%CE%BA%EE%C0%AE
# この名称であればVRChat SDKがAuto Detectしてくれる
keyname_prefix = 'vrc.v_'
keyname_suffices = [ 'sil', 'pp', 'aa', 'oh', 'ou', 'ih', 'th', 'nn', 'dd', 'kk', 'ff', 'e', 'ch', 'rr', 'ss' ]

# オブジェクトは一つだけ選択すること
if len(bpy.context.selected_objects) != 1:
    raise Exception('Select only one object.')

obj = bpy.context.selected_objects[0]

# ブレンドシェイプを追加
## ベースシェイプがすでにあるなら次の行はいらない
obj.shape_key_add(name='base')

## リップシンク用のシェイプキーを追加
for suf in keyname_suffices:
    obj.shape_key_add(name=keyname_prefix + suf)

```

## 8. VRMでエクスポート
VRMをチェックして，エラーがなければエクスポートします．
ウェイトが乗ってないよ警告は乗せないと変になるので乗せましょう．
マテリアルが違うよエラーはとりあえずMToonをつけておきましょう．結局Unityでまた調整するので．

## 9. Unityでインポート, VRChat向けに変換
あとはVRMのワークフローに従います．
注意点はリップシンクの設定をAuto Detectすることと，まばたきのブレンドシェイプを設定することです．

![Import to Unity](/assets/img/vrm-tutorial/15-Import_to_Unity_and_adjust_parameters.png)

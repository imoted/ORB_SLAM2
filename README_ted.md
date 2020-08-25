# 参考スライド

https://www.slideshare.net/takmin/20180424-orb-slam

* すごく基礎でわかりやすい
https://www.slideshare.net/MasayaKaneko/slamorbslam

* ORB特徴量
  * 画素ペアの大小でキーポイントと特徴量を計算するため、非 常に高速
  * キーポイントは画像ピラミッドの各レベルでFASTにより検出 
  * キーポイント周辺の画素ペアの値の大小を０と１に割り当て（バイナ リ特徴）
  * バイナリ特徴のため省メモリ 
  * 回転に対して不変 ORBで使用する画素ペア （学習された例）

* vocabraly
 place recongnition

* Tracking / Local Mapping / Loop Closing

* ORB_SLAM2はORB_SLAMをステレオカメラと、RGBD用に拡張したもの

* DBoW2 : Distributed Bag of Words
多数の文書の中から，文書IDを入力値に，その文書内からランダムに選択された単語を予測することで文書全体の意味を獲得する手法
Place recognitionに使用

* Features from Accelerated Segment Test (FAST)が提案されています。FASTは、機械学習により決定木を構築し、その決定木をトラバーサルすることで高速にコーナーを検出します。しかし、テクスチャが複雑な自然領域(木の葉，植え込み等)からコーナー点を多く検出する問題があります。このようなコーナー点はキーポイントの対応付けにおいては不要な点であり、キーポイントの対応付けの計算コストを増大します。
![](./pics/keypoint_web1.png)

* Structure from Motion (SfM)

画像を取得すると，そこには３次元空間を反映する様々な情報が含まれる。
画像に映った対象の３次元的形状を画像から得る方法として，影を用いる方法（Shape from Shading）や，ピントを利用する用いる方法（Shape from Defocus）などがあり，これらをま
とめたものが Shape from Xという概念である。

移動（Motion）するカメラから得られる画像から形状を復元するのが SfM（Shape from Motion）である。
Structure from Motion（SfM）は Shape from Motionの別名である。ただし，Structure from Motionというと，画像に映った対象物の幾何学形状とカメラの動きを同時に復元する手法という意味合いが強くなる。以下では SfM は Structure from Motionの略称とす
る。

SfM はもともとコンピュータビジョンやロボットビジョンからきた概念である。カメラの位置姿勢と対象の座標を取得するということであれば，写真測量における空中三角測量と SfM との違いはないように思えるかもしれない。ただし SfM は本質的にはセンサやコンピュータを用いて外界の情報を把握することにある。明確に定義されているわけではないが，基本的にSfM は“自動的”に処理を行うことを前提にしている。SfM はコンピュータやロボットの視覚として利用できるよう，極力処理が自動化されていることが本質的である。
ロボットビジョンでは周囲の３次元構造と自分の位置を推定する技術のことを SLAM（Simultaneous Localization and Mapping）と呼んでいる。SLAM にはレーザセンサを利用したものもあるが，画像を使用するもの（Visual SLAM）の計測原理は SfM と同じといってよい。

https://www.jstage.jst.go.jp/article/jsprs/55/3/55_206/_pdf

SfM を行う上で重要なテクニックは，一連の画像の間で自動的に対応点（同じ場所が移っている点）を抽出することである。動画像の場合，特徴点の移動が少ないので，離散的に撮影された画像よりはるかに対応点抽出が行いやすい。連続的に撮影された画像（動画像）から特徴点の移動を見出すにはオプティカルフローが利用される

* SfM 物体の三次元構造の復元
　正確性が大事

* Visual SLAM リアルタイムなカメラ姿勢推定
 速度が大事

* 位置推定 SLAM
![](./pics/SLAM1.png)
![](./pics/SLAM2.png)


# Installation memo

./build.sh
は問題なく完了


./build_ros.sh
をするとエラー

```
[rosbuild] Building package ORB_SLAM2
[rosbuild] Error from syntax check of ORB_SLAM2/manifest.xml

CMake Error at /opt/ros/kinetic/share/ros/core/rosbuild/private.cmake:78 (message):
  [rosbuild] Syntax check of ORB_SLAM2/manifest.xml failed; aborting
Call Stack (most recent call first):
  /opt/ros/kinetic/share/ros/core/rosbuild/public.cmake:174 (_rosbuild_check_manifest)
  CMakeLists.txt:4 (rosbuild_init)
```
rosbuildを使用しているので？　gloovy以前のbuildシステム？

appliedAI-Initiative/orb_slam_2_ros
を試用中。

* 以下見ると、kineticでもインストールしている例はあるが、、、

https://medium.com/@j.zijlmans/orb-slam-2052515bd84c
https://qiita.com/nnn112358/items/1087d5b0e4df5367f48b


# orb_slam2_ros　

* usb_cam usb_cam-test.launch
でテスト　OK

結果うまく、方向性とMAPが取得できている。

以下D435での難しさ。
おそらく早く動かしすぎて、止まった。
また天井はライトで白飛びや特徴点の少なさから、MAPが作りにくい。
またD435のUSB3.0ケーブルの不調から、飛びやすい。

のが原因。

→　ロボットで試して見たい。
 rasppi mouse?

CPU使用率 180%-260%ぐらい。
RAM 4%


## 問題点

* D435 mono / rgbdの両方で体験

* ２秒ぐらい経つと、point cloudが更新されなくなる。

* D reconfigure
  * reset_map
    マップを消去して新しく開始するには、trueに設定します。リセット後、パラメーターは自動的にfalseに更新されます。
  * min_num_kf_in_map  
    追跡が失われた後にマップがリセットされないようにする必要があるキーフレームの数。
  * min_observations_for_ros_map
    ポイントクラウドでキーポイントを公開する必要がある最小限の観測値の数。これはSLAM自体の動作にはまったく影響しません。

* PC再起動したら、下記がでて、system resetが繰り返されて、５回めで止まるようになった。
Depth Threshold (Close/Far Points): 0.588107
Enable localization only: false
New map created with 810 points
Track lost soon after initialisation, reseting...
System Reseting
Reseting Local Mapper... done
Reseting Loop Closing... done
Reseting Database... done
New map created with 846 points
Track lost soon after initialisation, reseting...
System Reseting
Reseting Local Mapper... done
Reseting Loop Closing... done
Reseting Database... done
New map created with 921 points
Track lost soon after initialisation, reseting...
System Reseting
Reseting Local Mapper... done
Reseting Loop Closing... done
Reseting Database... done
New map created with 652 points
Track lost soon after initialisation, reseting...
System Reseting
Reseting Local Mapper... done
Reseting Loop Closing... done
Reseting Database... done
Map point vector is empty!
New map created with 500 points

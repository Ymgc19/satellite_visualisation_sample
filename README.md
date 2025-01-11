# Sentinel-1の偏波VHを取得するGEEコード
Pythonのサンプルコード内で使用するデータを取得します

```
// ベトナムのメコン川付近のエリアを指定 
var region = ee.Geometry.Rectangle([105.29687496308843,10.480909588897706, 106.17028804902593,11.011146327642287]);

// Sentinel-1のデータをフィルター
var Sentinel1 = ee.ImageCollection('COPERNICUS/S1_GRD') // プロダクトを指定
                  .filterBounds(region) // 地域指定
                  .filter(ee.Filter.date('2024-04-10', '2024-04-30')) // 取得する画像の期間
                  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH')) //偏波VHを含むもの
                  .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) // 下降軌道
                  .filter(ee.Filter.eq('platform_number', 'A')) // ひとつの衛星のデータに限定
                  .filter(ee.Filter.eq('instrumentMode', 'IW')) //
                  .filter(ee.Filter.eq('resolution_meters', 10)) // 分解能
                  .select(['VH']); // 偏波VHを含むもの
print(Sentinel1); // 条件に合う画像の一覧を表示

// リストに変換
var imageList = Sentinel1.toList(Sentinel1.size());

// リスト内の各イメージをGoogle Driveにエクスポート
for (var i = 0; i < Sentinel1.size().getInfo(); i++) {
  var image = ee.Image(imageList.get(i));
  
  // 観測日を取得（ダウンロードするファイル名に日付を入れるため）
  var observationDate = image.date().format('yyyy-MM-dd').getInfo();
  var imageName = 'S1_VH_' + observationDate; // 観測日を名前に含める
  
  Export.image.toDrive({
    image: image,
    description: imageName,
    scale: 10,
    // folder: 'hoge', // 保存先フォルダを指定（必要に応じてコメント解除）
    region: region,
    crs: 'EPSG:4326' // 出力の座標系を指定
  });
}



// 可視化する
Map.centerObject(region);

// 最初の画像を指定
var firstImage = ee.Image(imageList.get(0)); // リスト内の最初の画像
Map.addLayer(firstImage, {min: -30, max: 0, palette: ['blue', 'yellow', 'red']}, 'polarisation VH');

```

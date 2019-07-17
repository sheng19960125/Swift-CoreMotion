# Swift-CoreMotion
## 什麼是CoreMotion？
Core Motion 來自ios硬件的運動與環境相關數據，包含來自加速度、陀螺儀、計步器、氣壓計等等。我們可以使用此框架訪問硬件等生成的數據，我們可以利用iPhone、iPad、apple watch等設備讀取處理來自傳感器的數據，不會造成CPU等負擔或耗電量。

## 什麼是CMMotionManager?
CMMotionManager用在啟動管理運動感測檢測到的數據服務，次物件可以接收四種類型的數據資料。  
  1.測量加速度，或隨著時間的速度變化量。  
  2.測得目前陀螺儀的狀態，設備的方向等。  
  3.磁力儀基本上可以稱作為指南針，相對於地球磁場。  
  4.除了可以單獨呼叫外，我們更可以統一上述幾種傳感器，整合成自己所需的數據。  

## Example  CMMotionManager
```
import CoreMotion`  
import UiKit
class ViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    //個別設定個別所需的參數
    //獲取陀螺儀、加速度數據的非重力數據部分來添加新的運動方法
    
    //manager:X軸方向加速度超過2.5gs的話就進行運動，manager2也是利用此方法進行運算
    var manager = CMMotionManager()
    var manager2 = CMMotionManager()
    //通過統一設備運動數據來合成陀螺儀與加速度計數據，CoreMotion將重力與加速度分開並將每個移動量做為圖片的屬性值。
    var manager3 = CMMotionManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView.image = UIImage(named: "image.jpg")!
        
        //確認手機是否有支持
        guard manager.isAccelerometerAvailable else {
            return
        }
        
        self.myDeviceMotion()
        self.myDeviceMotionPop()
        self.myAccelerometerMotion()
    }
    

  
    func myDeviceMotion(){
        manager.deviceMotionUpdateInterval = 0.05
        manager.startDeviceMotionUpdates(to: OperationQueue.current!) { (data, error) in
            
            guard let data = data , error == nil else {
                return
            }
            if(data.userAcceleration.y < -3.0){
                self.alertCheckMotionAction("上下 success")
            }
        }
    }
    
    /*
     * 此部分結合陀螺儀/加速度數據的其他非重力部分來添加新的交互方法，
     * 設定瞬間0.01Ｓ的運動狀態捕捉，偏移量。
     */
    func myDeviceMotionPop(){
        manager2.deviceMotionUpdateInterval = 0.01
        manager2.startDeviceMotionUpdates(to: OperationQueue.current!) {
            (data, error) in
            guard let data = data , error == nil else {
                return
            }
            if data.userAcceleration.x < -2.5 {
                self.alertCheckMotionAction("左側 success")
            }
        }
    }
    
    //圖片不隨個手機旋轉
    //atan2 -> 算圓周pi多少 半圈=3.1415926....
    //第一象限至第二象限 由3.14 -> 0, 第三象限至第四象限 由0 -> -3.14....
    func myAccelerometerMotion(){
        manager3.deviceMotionUpdateInterval = 0.01
        manager3.startDeviceMotionUpdates(to: OperationQueue.current!) {
            (data, error) in
            guard let data = data , error == nil else { return }
            
            let rotation = atan2(data.gravity.x, data.gravity.y) - .pi
            let rotationAngle = CGFloat(rotation)
            self.imageView.transform = CGAffineTransform(rotationAngle: rotationAngle)
        }
        
    }
    
    func alertCheckMotionAction(_ message:String){
        // 建立一個提示框
        let alertController = UIAlertController(title: "", message: message, preferredStyle: .alert)
        let okAction = UIAlertAction(title: "OK", style: .default, handler: {
                (action: UIAlertAction!) -> Void in
                print("按[確認]，執行 clouser....")
        })
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
}
```
## 補充
Swift 內建的Api接口可直接調用晃動的動作，例如Wechat裡面的搖一搖。
```
    override func motionBegan(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        print("開始搖晃")
    }
    
    override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        print("搖晃結束")
        if(event?.subtype == UIEvent.EventSubtype.motionShake){
            print("do something")
        }
    }
    
    override func motionCancelled(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        print("搖晃意外終止")
    }
```

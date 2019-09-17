# TDD实例

项目中对于 `TDD` 的实战，依赖的是 `GoogleTest` 框架

我负责编码单元对中控提供
* 设置编码单元
* 设置视频源
* 设置视频输出
* 状态检测
* 开启通道
* 关闭通道

这 6 个接口，中控通过 `http` 调用编码单元接口，为了解耦和方便进行 `TDD` 测试，我们将这几个方法写成一个抽象类，编码单元再实现具体的方法：
```C++
class IEventHandler {
 public:

  virtual bool StartChannel(int channel_id) = 0;

  virtual bool StopChannel(int channel_id) = 0;

  virtual bool ConfigChannel(int channel_id,
                             const tucodec::SChannelSourceConfig &source_config) = 0;

  virtual bool ConfigChannel(int channel_id,
                             const tucodec::SChannelDestConfig &dest_config) = 0;

  virtual bool ConfigChannel(int channel_id,
                             const tucodec::SChannelEncoderConfig &encoder_config) = 0;

  virtual void DetectWorker(int channel_id, int duration,
                             tucodec::SWorkerStatus &worker_status) = 0;

};
```
这里 `ConfigChannel` 方法处理不同的参数结构体。

在测试用例中首先继承 `testing::Test` 和 `public IEventHandler`
```C++

class HttpControllerTest : public testing::Test, public IEventHandler {
 public:

  bool StartChannel(int channel_id) override {
    channel_id_ = channel_id;    
    return true;
  }

  bool StopChannel(int channel_id) override {
    channel_id_ = channel_id;  
    return true;
  }

  bool ConfigChannel(int channel_id,
                     const tucodec::SChannelSourceConfig &source_config) override {
    channel_id_ = channel_id;
    // 其他参数省略
    source_config_.username = source_config.username;

    return true;
  }

  bool ConfigChannel(int channel_id,
                     const tucodec::SChannelDestConfig &dest_config) override {
    channel_id_ = channel_id;
    // 其他参数省略
    dest_config_.port = dest_config.port;
    return true;
  }

  bool ConfigChannel(int channel_id,
                     const tucodec::SChannelEncoderConfig &encoder_config) override {
    channel_id_ = channel_id;
    // 其他参数省略
    encoder_config_.output_h = encoder_config.output_h;  
    return true;
  }

  void DetectWorker(int channel_id, int duration,
                            tucodec::SWorkerStatus &worker_status) override {
    channel_id_ = channel_id;
  };  

 protected:
  void SetUp() override {    
    tucodec::TuLog::Init("", "http_controller_test.log", 50, 2 * 1024 * 1024);
    tucodec::TuLog::Instance()->SetPriority(tucodec::kTulogDebug);   
    controller_.Init();
  }

  void TearDown() override {  
    tucodec::TuLog::Destory();
  }
 protected :  
  HttpController controller_{this};
  tucodec::SChannelSourceConfig source_config_;
  tucodec::SChannelDestConfig dest_config_;
  tucodec::SChannelEncoderConfig encoder_config_;
  int channel_id_{};
};

TEST_F(HttpControllerTest, GetStateTest) {

  unsigned char get_json[] = "{\n"
                               "    \"cmd\":\"get_state\",\n"
                               "    \"chn_id\":1,\n"
                               "    \"duration\":2\n"
                               "}";
  controller_.ResolveData(get_json, sizeof(get_json));

  ASSERT_EQ(channel_id_, 1);
}


TEST_F(HttpControllerTest, StartChannelTest) {

  unsigned char start_json[] = "{\n"
                               "    \"cmd\":\"start_chn\",\n"
                               "    \"chn_id\":1\n"
                               "}";
  controller_.ResolveData(start_json, sizeof(start_json));

  ASSERT_EQ(channel_id_, 1);
}

TEST_F(HttpControllerTest, StopChannelTest) {

  unsigned char stop_json[] = "{\n"
                              "    \"cmd\":\"stop_chn\",\n"
                              "    \"chn_id\":2\n"
                              "}";

  controller_.ResolveData(stop_json, sizeof(stop_json));

  ASSERT_EQ(channel_id_, 2);
}

TEST_F(HttpControllerTest, DestConfigTest) {

  unsigned char destination_json[] = "{\n"
                                     "    \"cmd\":\"set_destination\",\n"
                                     "    \"chn_id\":1,\n"
                                     "    \"address\":\"192.168.1.221\",\n"
                                     "    \"port\":554,\n"
                                     "    \"destination_type\":\"onvif\",\n"
                                     "    \"stream_id\":\"live1\"\n"
                                     "}";
  controller_.ResolveData(destination_json, sizeof(destination_json));

  ASSERT_EQ(channel_id_, 1);
  ASSERT_EQ(dest_config_.address, "192.168.1.221");
  ASSERT_EQ(dest_config_.port, 554);
  ASSERT_EQ(dest_config_.destination_type, "onvif");
  ASSERT_EQ(dest_config_.stream_id, "live1");
}
```
实现具体接口，这里只需要对他进行赋值即可，将本身传入 `HttpController` 中，这里我们先不测试网络，直接调用接收到 `http` 请求后解析 `json` 的函数。再根据参数中命令选择调用 设置编码源还是具体哪一个接口。因为需要在回调函数中解析 `json`，这里就直接继承 `DataCallBack` 类，在接收方法中直接调用 `ResolveData` 。
```C++
class HttpController : public tucodec::DataCallBack {
 public:
  explicit HttpController(IEventHandler* event_handler);
  virtual ~HttpController();

  bool Init(const std::string &server_ip = "0.0.0.0", int server_port = 8081);
  bool ResolveData(unsigned char* data, unsigned int len);

  void ReceiveDataCallBack(int local_busy_handle,
                           int remote_handle,
                           unsigned char* data,
                           unsigned int data_length,
                           void* user) override {
  };
  void ReceiveDataCallBack(int local_busy_handle,
                           const std::string &remote_ip,
                           int remote_port,
                           unsigned char* data,
                           unsigned int data_length,
                           void* user) override {

  };
  void ReceiveDataCallBack(int local_busy_handle,
                                   const std::string& remote_ip,
                                   int remote_port,
                                   const std::string& path,
                                   unsigned char* data,
                                   unsigned int data_length,
                                   void* user) override {
    tucodec::TuLog::Instance()->Info("receive data: %s length: %d ", data, data_length); //NOLINT
    tucodec::Server* server = reinterpret_cast<tucodec::Server*>(user);


    if (ResolveData(data, data_length)) {
      unsigned char response[] = "{\"code\":0}";
      int len = sizeof(response);
      server->SendResponse2Client(0, response, len);
    } else {
      unsigned char response[] = "{\"code\":-1}";
      int len = sizeof(response);
      server->SendResponse2Client(0, response, len);
    }

  };
  void StatusCallBack(int local_busy_handle,
                      int remote_handle,
                      std::string remote_ip,
                      int remote_port,
                      int remote_status,
                      void* user) override {

  };
 private:
  IEventHandler* event_handler_;
  tucodec::Server* server_;
};
```
```C++
const char *k_set_codec_source = "set_codec_source";
const char *k_set_destination = "set_destination";
const char *k_start_chn = "start_chn";
const char *k_stop_chn = "stop_chn";
const char *k_get_state = "get_state";
const char *k_service_state = "service_state";

HttpController::HttpController(IEventHandler *event_handler) {
  event_handler_ = event_handler;
  server_ = nullptr;
}

HttpController::~HttpController() {
  if (server_ != nullptr) {
    delete server_;
    server_ = nullptr;
  }
}

bool HttpController::Init(const std::string &server_ip, int server_port) {
  return true;
}

bool HttpController::ResolveData(unsigned char *data, unsigned int len) {

  tucodec::TuJson json;
  std::stringstream x_read_write;
  x_read_write.write((const char *) data, len);
  json.Load(x_read_write);

  std::string cmd = json.GetString("cmd");
  int channel_id = json.GetInt("chn_id");

  if (cmd == k_set_codec_source) {
    SourceCodecHandler source_codec_handler;
    return source_codec_handler.HandleJson(json, channel_id, event_handler_);
  }
  if (cmd == k_set_destination) {
    DestinationHandler destination_handler;
    return destination_handler.HandleJson(json, channel_id, event_handler_);
  }
  if (cmd == k_get_state) {
    GetStatusHandler get_status_handler;
    return get_status_handler.HandleJson(json, channel_id, event_handler_);
  }
  if (cmd == k_start_chn) {
    return event_handler_->StartChannel(channel_id);
  }
  if (cmd == k_stop_chn) {
    return event_handler_->StopChannel(channel_id);
  }

  return false;
}
```
这边还添加了一个类专门用来处理不同 `json`，方便以后扩展
```C++
class IHttpHandler {
 public:
  virtual bool HandleJson(tucodec::TuJson &json, int channel_id, //NOLINT
                          IEventHandler* event_handler_) = 0; //NOLINT
};

class DestinationHandler : public IHttpHandler {
 public:
  bool HandleJson(tucodec::TuJson &json, int channel_id,
                  IEventHandler* event_handler_) override;
};

class SourceCodecHandler : public IHttpHandler {
 public:
  bool HandleJson(tucodec::TuJson &json, int channel_id,
                  IEventHandler* event_handler_) override;
};

class GetStatusHandler : public IHttpHandler {
 public:
  bool HandleJson(tucodec::TuJson &json, int channel_id,
                  IEventHandler* event_handler_) override;
};
```
```C++

bool DestinationHandler::HandleJson(tucodec::TuJson &json,
                                    int channel_id, IEventHandler *event_handler_) { //NOLINT
  if (event_handler_ == nullptr) {
    return false;
  }
  tucodec::ChannelDestConfig channel_dest_config;
  channel_dest_config.port = json.GetInt("port");
  channel_dest_config.address = json.GetString("address");
  channel_dest_config.stream_id = json.GetString("stream_id");
  channel_dest_config.destination_type = json.GetString("destination_type");
  return event_handler_->ConfigChannel(channel_id, channel_dest_config);
}
bool SourceCodecHandler::HandleJson(tucodec::TuJson &json,
                                    int channel_id, IEventHandler *event_handler_) { //NOLINT
  if (event_handler_ == nullptr) {
    return false;
  }
  // codec
  tucodec::ChannelEncoderConfig channel_encoder_config;
  // 省略部分参数
  channel_encoder_config.output_w = json.GetInt("output_w");

  if (!event_handler_->ConfigChannel(channel_id, channel_encoder_config)) {
    tucodec::TuLog::Instance()->Error("EncoderConfig Error!");
    return false;
  }

  // source
  tucodec::ChannelSourceConfig channel_source_config;
  channel_source_config.source_type = json.GetString("source_type");
  channel_source_config.address = json.GetString("address");
  channel_source_config.port = json.GetInt("port");
  channel_source_config.username = json.GetString("username");
  channel_source_config.password = json.GetString("password");
  return event_handler_->ConfigChannel(channel_id, channel_source_config);;
}

bool GetStatusHandler::HandleJson(tucodec::TuJson &json, int channel_id, IEventHandler *event_handler_) {
  if (event_handler_ == nullptr) {
    return false;
  }
  tucodec::SWorkerStatus status;
  int  duration = json.GetInt("duration");
  event_handler_->DetectWorker(channel_id, duration, status);
  return status.encoder_working == 1 && status.source_working == 1;
}
```
解析接口测试完毕后，接下来加入网络模块。
在 `HttpController` 类中先添加成员变量 `tucodec::Server *server_;`，修改构造和析构
```C++
HttpController::HttpController(IEventHandler *event_handler) {
  event_handler_ = event_handler;
  server_ = nullptr;
}

HttpController::~HttpController() {
  if (server_ != nullptr) {
    delete server_;
    server_ = nullptr;
  }
}
```
`Init` 方法中添加：
```C++
if (server_ == nullptr) {
  server_ = tucodec::NetWorkFactory::CreateHttpServer(
      server_ip,
      server_port);
  server_->SetDataCallBack(this,
                           reinterpret_cast<void *>(server_));
  if (!server_->StartServer()) {
    tucodec::TuLog::Instance()->Error("start http server failed!");
    return false;
  }
}
tucodec::TuLog::Instance()->Info("start http server success!");
```
测试类中加入客户端初始化以及阻塞函数：
```C++
void Wait(int time) {
  while (watcher_ == 0 && time > 0) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    --time;
  }
}

protected:
void SetUp() override {
  watcher_ = 0;
  tucodec::TuLog::Init("", "http_controller_test.log", 50, 2 * 1024 * 1024);
  tucodec::TuLog::Instance()->SetPriority(tucodec::kTulogDebug);

  controller_.Init();

  client_ = tucodec::NetWorkFactory::CreateHttpClient(
      "apt/v1/test",
      tucodec::kRequestPost);
  client_->ConnectServer("127.0.0.1", 8081);
  client_->SetDataCallBack(nullptr, nullptr);
}

void TearDown() override {
  delete client_;
  client_ = nullptr;
  tucodec::TuLog::Destory();
}
protected :
tucodec::Client *client_ = nullptr;
HttpController controller_{this};
int watcher_{};
```
添加测试用例：
```C++

TEST_F(HttpControllerTest, SourceCodeConfigTest) {

  unsigned char source_codec_json[] = "{\n"
                                      "    \"cmd\":\"set_codec_source\",\n"
                                      "    \"chn_id\":1,\n"
                                      "    \"source_type\":\"onvif\"\n"
                                      "}";

  client_->SendRequest2Server(source_codec_json, sizeof(source_codec_json));
  Wait(2);

  // 省略参数
  ASSERT_EQ(channel_id_, 1);
  ASSERT_EQ(source_config_.source_type, "onvif");
}
```
尽量解耦和，让业务依赖于接口，而非接口依赖于业务，每一个小模块都可以单独测试，在保证程序按预期稳定运行的基础上(有测试用例的存在)，再将我们的代码进行重构，小步前进，出现问题也能快速定位，或者 `revert` 。

****

我们这里用的是 `TestCase` 级别的事件机制

`gtest` 提供了多种事件机制，非常方便我们在案例之前或之后做一些操作。总结一下 `gtest` 的事件一共有 3 种：
1. 全局的，所有案例执行前后。
2. `TestSuite` 级别的，在某一批案例中第一个案例前，最后一个案例执行后。
3. `TestCase` 级别的，每个 `TestCase` 前后。

### 全局事件

要实现全局事件，必须写一个类，继承 `testing::Environment` 类，实现里面的 `SetUp` 和 `TearDown` 方法。
1. `SetUp()` 方法在所有案例执行前执行
2. `TearDown()` 方法在所有案例执行后执行

```C++
class FooEnvironment : public testing::Environment
{
public:
    virtual void SetUp()
    {
        std::cout << "Foo FooEnvironment SetUP" << std::endl;
    }
    virtual void TearDown()
    {
        std::cout << "Foo FooEnvironment TearDown" << std::endl;
    }
};
```
当然，这样还不够，我们还需要告诉 `gtest` 添加这个全局事件，我们需要在 `main` 函数中通过 `testing::AddGlobalTestEnvironment` 方法将事件挂进来，也就是说，我们可以写很多个这样的类，然后将他们的事件都挂上去。

```C++
int main(int argc, char* argv[])
{
    testing::AddGlobalTestEnvironment(new FooEnvironment);
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

### TestSuite事件

我们需要写一个类，继承 `testing::Test` ，然后实现两个静态方法
1. `SetUpTestCase()` 方法在第一个 `TestCase` 之前执行
2. `TearDownTestCase()` 方法在最后一个 `TestCase` 之后执行

```C++
class FooTest : public testing::Test {
 protected:
  static void SetUpTestCase() {
    shared_resource_ = new ;
  }
  static void TearDownTestCase() {
    delete shared_resource_;
    shared_resource_ = NULL;
  }
  // Some expensive resource shared by all tests.
  static T* shared_resource_;
};
```
在编写测试案例时，我们需要使用 `TEST_F` 这个宏，第一个参数必须是我们上面类的名字，代表一个 `TestSuite` 。
```C++
TEST_F(FooTest, Test1)
 {
    // you can refer to shared_resource here
}
TEST_F(FooTest, Test2)
 {
    // you can refer to shared_resource here
}
```

### TestCase事件
`TestCase` 事件是挂在每个案例执行前后的，实现方式和上面的几乎一样，不过需要实现的是 `SetUp` 方法和 `TearDown` 方法：
1. `SetUp()` 方法在每个 `TestCase` 之前执行
2. `TearDown()` 方法在每个 `TestCase` 之后执行

```C++
class FooCalcTest:public testing::Test
{
protected:
    virtual void SetUp()
    {
        m_foo.Init();
    }
    virtual void TearDown()
    {
        m_foo.Finalize();
    }

    FooCalc m_foo;
};

TEST_F(FooCalcTest, HandleNoneZeroInput)
{
    EXPECT_EQ(4, m_foo.Calc(12, 16));
}

TEST_F(FooCalcTest, HandleNoneZeroInput_Error)
{
    EXPECT_EQ(5, m_foo.Calc(12, 16));
}
```
在 `TEST_F` 中使用的变量可以在初始化函数SetUp中初始化，在 `TearDown` 中销毁，并且所有的 `TEST_F` 是互相独立的，都是在初始化以后的状态开始运行，一个 `TEST_F` 不会影响另一个 `TEST_F` 所使用的数据

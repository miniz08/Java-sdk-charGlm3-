需要的依赖：
```
    // OkHttp 依赖
    implementation 'com.squareup.okhttp3:okhttp:4.10.0'

    // Gson 依赖
    implementation 'com.google.code.gson:gson:2.9.0'
```
直接打开文件然后把代码粘到你的ide里,然后把依赖粘过去，更新依赖之后就能运行
只是一个响应示例，实际运用需要结合场景
不想打开文件直接从这里粘也可以(雾)
```
package org.example;
import okhttp3.*;
import com.google.gson.*;
import java.io.IOException;
import java.util.*;

public class ChatbotClient {

    private static final String API_KEY = "";  // 替换为你的API密钥
    private static final String API_URL = "https://open.bigmodel.cn/api/paas/v4/chat/completions";  // ZhipuAI的API地址
    private static OkHttpClient client = new OkHttpClient();

    // 模拟聊天历史
    private static List<Map<String, String>> chatHistory = new ArrayList<>();

    public static void main(String[] args) {
        String userMessage = "二小姐";  // 用户输入的消息
        try {
            String response = getChatResponse(userMessage);
            System.out.println("芙兰: " + response);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 获取AI的聊天回复
    public static String getChatResponse(String userMessage) throws IOException {
        // 将用户消息添加到聊天历史
        Map<String, String> userMessageMap = new HashMap<>();
        userMessageMap.put("role", "user");
        userMessageMap.put("content", userMessage);
        chatHistory.add(userMessageMap);

        // 创建请求体
        Map<String, Object> requestBodyMap = new HashMap<>();
        requestBodyMap.put("model", "charglm-3");  // 模型名称
        requestBodyMap.put("messages", chatHistory);

        // 附加meta信息
        Map<String, String> meta = new HashMap<>();
        meta.put("user_info", "误入幻想乡的人类");
        meta.put("bot_info", "芙兰朵露是495岁的吸血鬼，魔法少女。她是红魔馆的二小姐，蕾米莉亚的妹妹。萝莉体型，身穿鲜红色的洋装，是一个不谙世事，有些孤独的少女" +
                "具有破坏一切程度的能力，深知自己有着强大的力量，对于一切在自己之下的物种有着不自觉的轻视，实际上是个非常可爱的女孩" +
                "她平常说话天真，总是不自觉的装出有威严的样子");
        meta.put("bot_name", "芙兰朵露·斯卡雷特");
        meta.put("user_name", "人类");
        requestBodyMap.put("meta", meta);

        // 使用Gson将请求体转换为JSON
        Gson gson = new Gson();
        String jsonRequest = gson.toJson(requestBodyMap);

        // 创建POST请求
        RequestBody body = RequestBody.create(MediaType.parse("application/json"), jsonRequest);
        Request request = new Request.Builder()
                .url(API_URL)
                .addHeader("Authorization", "Bearer " + API_KEY)  // 用API密钥授权
                .post(body)
                .build();

        // 执行请求并处理响应
        Response response = client.newCall(request).execute();
        if (response.isSuccessful()) {
            String jsonResponse = response.body().string();
            JsonObject jsonObject = JsonParser.parseString(jsonResponse).getAsJsonObject();
            String replyContent = jsonObject.getAsJsonArray("choices")
                    .get(0).getAsJsonObject()
                    .getAsJsonObject("message")
                    .get("content").getAsString();

            // 将AI回复添加到历史聊天记录
            Map<String, String> assistantMessageMap = new HashMap<>();
            assistantMessageMap.put("role", "assistant");
            assistantMessageMap.put("content", replyContent);
            chatHistory.add(assistantMessageMap);

            return replyContent;
        } else {
            throw new IOException("Request failed with status code: " + response.code());
        }
    }
}
```

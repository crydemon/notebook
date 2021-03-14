~~~
public class BIOServer {

  static class Handler implements Runnable {

    private Socket socket;

    public Handler(Socket socket) {
      this.socket = socket;
    }

    @Override
    public void run() {
      try {
        BufferedWriter bw = new BufferedWriter(
            new OutputStreamWriter(socket.getOutputStream(), StandardCharsets.UTF_8));
        BufferedReader br = new BufferedReader(
            new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8));

        char[] buffer = new char[1024];
        StringBuilder content = new StringBuilder();
        int i;
        while ((i = br.read(buffer)) != -1) {
          content.append(buffer, 0, i);
          if (content.toString().contains("end")) {
            break;
          }
        }
        System.out.println(content);
        bw.write("fuck you");
        bw.flush();
        bw.close();
        br.close();
      } catch (IOException e) {
        e.printStackTrace();
      } finally {

      }
    }
  }

  public static void main(String[] args) {
    try {
      ServerSocket serverSocket = new ServerSocket(9999);
      Socket socket;
      ExecutorService executorService = new ThreadPoolExecutor(10, 10, 60L, TimeUnit.SECONDS,
          new ArrayBlockingQueue(10));
      while (true) {
        socket = serverSocket.accept();
        Handler handler = new Handler(socket);
        executorService.execute(handler);
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

}

~~~


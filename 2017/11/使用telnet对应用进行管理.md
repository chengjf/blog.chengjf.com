# 使用telnet对应用进行管理

服务端程序启动后，要向外提供服务。如果要对服务端程序进行操作，比如切量（从注册中心注册/退出，停止/开始去接收/发送kafka消息），获取服务信息（当前启动的service
列表，消息队列情况）等等。常用的操作是做一个服务端管理系统进行管理，常见的是做一个web系统，比如motan，druid等。但是做web系统可能很麻烦耗时，这里介绍使用telnet进行管理。

telnet是几乎所有操作系统带有的命令，使用广泛。那使用telnet对应用进行管理，就是监听一个端口，处理telnet连接即可。


关键点：
1. 实现***AdminCommand***接口，定制自己的命令
2. 使用***registerCommand***注册自己的命令

> help或？命令输出当前AdminServer所有的命令

> 和Spring集成的时候，和Spring启动在一起，获得ApplicationContext，这样就可以获得Spring容器中的所有对象，调用更加方便。

```java

import java.io.*;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.regex.Matcher;
import java.util.regex.Pattern;


public class AdminServer implements Runnable {
    boolean keepAlive;
    String prompt;
    String encoding;
    private List<AdminServer.CmdInfo> cmdinfos;
    private Map<String, AdminCommand> cmds;
    private int port;
    private String address;
    private ExecutorService service;

    public AdminServer(String address, int port) {
        this(address, port, true);
    }

    public AdminServer(String address, int port, boolean keepAlive) {
        this(address, port, keepAlive, "admin> ");
    }

    public AdminServer(String address, int port, boolean keepAlive, String prompt) {
        this(address, port, keepAlive, prompt, "GBK");
    }

    public AdminServer(String address, int port, boolean keepAlive, String prompt, String encoding) {
        this.keepAlive = true;
        this.prompt = "";
        this.encoding = "GBK";
        this.cmdinfos = new CopyOnWriteArrayList();
        this.cmds = new ConcurrentHashMap();
        this.address = address;
        this.port = port;
        this.registerCommand("quit,q", new AdminCommand() {
            public void execute(String[] argv, PrintWriter out) {
                out.flush();
                out.close();
            }

            public String toString() {
                return "close this connection.";
            }
        });
        this.registerCommand("help,?", new HelpCommand(this.cmdinfos, this.cmds));
        this.keepAlive = keepAlive;
        this.prompt = prompt;
        this.encoding = encoding;
        this.service = Executors.newFixedThreadPool(1);
    }

    private static Pair<String, String> parseCmd(String fullName) {
        String[] ns = split(fullName, ",");
        return ns.length <= 1 ? Pair.makePair(fullName, (String) null) : Pair.makePair(ns[0], ns[1]);
    }

    public static void main(String[] argv) {
        AdminServer as = new AdminServer("0.0.0.0", 12345, true, "admin> ");
        as.registerCommand("test", new AdminCommand() {
            public void execute(String[] argv, PrintWriter out) {

                out.println("test argv: " + Arrays.toString(argv));
                int tryTimes = 3;
                int n = 0;
                while (true) {
                    if (n == 0) {
                        if (tryTimes == 0) {
                            break;
                        }
                        tryTimes--;
                    }
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                out.println("stop " + Arrays.toString(argv));

            }

            public String toString() {
                return "test command.\nusage:\n    test [option]";
            }
        });
        as.run();
    }

    public static String[] split(String line, String separator) {
        if (line != null && separator != null && separator.length() != 0) {
            ArrayList<String> list = new ArrayList();
            int pos1 = 0;

            while (true) {
                int pos2 = line.indexOf(separator, pos1);
                if (pos2 < 0) {
                    list.add(line.substring(pos1));

                    for (int i = list.size() - 1; i >= 0 && list.get(i).length() == 0; --i) {
                        list.remove(i);
                    }

                    return list.toArray(new String[0]);
                }

                list.add(line.substring(pos1, pos2));
                pos1 = pos2 + separator.length();
            }
        } else {
            return null;
        }
    }

    public AdminServer registerCommand(String name, AdminCommand cmd) {
        Pair<String, String> pr = parseCmd(name);
        this.cmdinfos.add(new AdminServer.CmdInfo(name, cmd));
        this.cmds.put(pr.first, cmd);
        if (pr.second != null) {
            this.cmds.put(pr.second, cmd);
        }

        return this;
    }

    public AdminCommand getCommand(String name) {
        return this.cmds.get(name);
    }


    public void run() {
        try {
            ServerSocket serverSocket = new ServerSocket();
            serverSocket.setReuseAddress(true);
            InetSocketAddress addr = new InetSocketAddress(InetAddress.getByName(this.address), this.port);
            serverSocket.bind(addr, 50);

            System.out.println("AdminServer start at port " + this.port);
            while (true) {
                Socket socket = null;

                try {
                    socket = serverSocket.accept();
                    service.submit(new AdminServer.AdminConnection(this, socket));
//                    new AdminServer.AdminConnection(this, socket).run();
                } catch (Exception e) {
                    System.err.println(e);
                }
            }
        } catch (Exception e) {
            System.err.println(this.address + ":" + this.port + " " + e);
        }
    }

    public interface AdminCommand {

        void execute(String[] argv, PrintWriter out);
    }

    static final class AdminConnection implements Runnable {
        private static final Pattern pat = Pattern.compile("\\s+(([^'\\t\\r\\n]+)|('[^'\\r\\n]*')|(\\\"[^\\r\\n]*))");
        private Socket socket;
        private AdminServer as;

        public AdminConnection(AdminServer as, Socket socket) {
            this.as = as;
            this.socket = socket;
        }

        private static Pair<String, String[]> parseCmd(String cmdLine) {
            cmdLine = cmdLine.trim();
            Matcher m = pat.matcher(cmdLine);

            ArrayList argv;
            String cmd;
            for (argv = new ArrayList(); m.find(); argv.add(cmd)) {
                cmd = m.group(1);
                char c1 = cmd.charAt(0);
                char c2 = cmd.charAt(cmd.length() - 1);
                if (c1 == '\'' && c2 == '\'' || c1 == '"' && c2 == '"') {
                    cmd = cmd.substring(1, cmd.length() - 1);
                }
            }

            cmd = cmdLine;
            int n = cmdLine.indexOf(32);
            if (n > 0) {
                cmd = cmdLine.substring(0, n);
            }

            String[] argvs = new String[argv.size()];
            argvs = (String[]) argv.toArray(argvs);
            return Pair.makePair(cmd, argvs);
        }

        @Override
        public void run() {
            try (BufferedReader in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()))) {

                try (PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(this.socket.getOutputStream(), this.as.encoding)))
                ) {
                    do {
                        if (this.as.prompt.length() > 0) {
                            out.print(this.as.prompt);
                            out.flush();
                        }

                        String cmdLine = in.readLine();
                        if (cmdLine == null) {
                            return;
                        }

                        if (cmdLine.length() != 0) {
                            Pair<String, String[]> pr = parseCmd(cmdLine);
                            AdminCommand c = this.as.getCommand(pr.first);
                            if (c != null) {
                                c.execute(pr.second, out);
                            } else {
                                out.println("Invalid command. Please type help or help you by smartlv.");
                            }

                            out.flush();
                        }
                    } while (this.as.keepAlive);
                }
            } catch (IOException e) {
                System.err.println(e);
            }
        }
    }

    static class CmdInfo {
        String name;
        AdminCommand cmd;

        CmdInfo(String name, AdminCommand cmd) {
            this.name = name;
            this.cmd = cmd;
        }
    }

    static class HelpCommand implements AdminCommand {
        private static final String SPACE_FILL = getSpace(20);
        private List<CmdInfo> cmdInfos;
        private Map<String, AdminCommand> cmds;

        public HelpCommand(List<CmdInfo> cmdInfos, Map<String, AdminCommand> cmds) {
            this.cmdInfos = cmdInfos;
            this.cmds = cmds;
        }

        private static String getSpace(int count) {
            if (count <= 0) {
                return " ";
            } else {
                StringBuilder sb = new StringBuilder();

                for (int i = 0; i < count; ++i) {
                    sb.append(" ");
                }

                return sb.toString();
            }
        }

        public void execute(String[] argv, PrintWriter out) {
            if (argv.length > 0) {
                String name = argv[0];
                AdminCommand c = this.cmds.get(name);
                if (c == null) {
                    out.println("can not find command: " + name);
                } else {
                    out.println(name);
                    out.println("---------");
                    out.println(c);
                }

            } else {
                out.println("admin commands: ");
                Iterator iterator = this.cmdInfos.iterator();

                while (iterator.hasNext()) {
                    CmdInfo c = (CmdInfo) iterator.next();
                    out.print(c.name);
                    out.print(getSpace(20 - c.name.length()));
                    String[] lines = split(c.cmd.toString(), "\n");
                    out.println(lines[0]);

                    for (int i = 1; i < lines.length; ++i) {
                        out.print(SPACE_FILL);
                        out.println(lines[i]);
                    }
                }

            }
        }

        public String toString() {
            return "print admin command help.\nusage:\n\thelp [cmd]";
        }
    }

    static class Pair<F, S> implements Serializable {
        private static final long serialVersionUID = 1L;
        public F first;
        public S second;

        public Pair() {
        }

        public Pair(F f, S s) {
            this.first = f;
            this.second = s;
        }

        public static <FT, ST> Pair<FT, ST> makePair(FT f, ST s) {
            return new Pair(f, s);
        }

        private static <T> boolean eq(T o1, T o2) {
            return o1 == null ? o2 == null : o1.equals(o2);
        }

        private static int h(Object o) {
            return o == null ? 0 : o.hashCode();
        }

        public boolean equals(Object o) {
            Pair<F, S> pr = (Pair) o;
            if (pr == null) {
                return false;
            } else {
                return eq(this.first, pr.first) && eq(this.second, pr.second);
            }
        }

        public int hashCode() {
            int seed = h(this.first);
            seed ^= h(this.second) + -1640531527 + (seed << 6) + (seed >> 2);
            return seed;
        }

        public F getFirst() {
            return this.first;
        }

        public void setFirst(F first) {
            this.first = first;
        }

        public S getSecond() {
            return this.second;
        }

        public void setSecond(S second) {
            this.second = second;
        }

        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append("{").append(this.first).append(", ").append(this.second).append("}");
            return sb.toString();
        }
    }
}


```
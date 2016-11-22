# folder-8

Program C#

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Windows.Forms;
using System.Threading;
using System.Net;
using System.Net.Sockets;



namespace WindowsFormsApplication1
{
    public partial class UDP_Send_Receive : Form
    {
        Thread thdUDPServer;
        public UDP_Send_Receive()
        {
            InitializeComponent();
        }
        private void groupBox1_Enter(object sender, EventArgs e)
        {
            
        }
        public void serverThread()
        {
            UdpClient udpClient = new UdpClient(2000);
            while (true)
            {
                IPEndPoint RemoteIpEndPoint = new IPEndPoint(IPAddress.Any, 0);
                Byte[] receiveBytes = udpClient.Receive(ref RemoteIpEndPoint);
                string returnData = Encoding.ASCII.GetString(receiveBytes);
                Invoke(new Action(() => textBoxTemp.Text = returnData + " Celcius"));
            }
        }


        private void buttonLedOn_Click_1(object sender, EventArgs e)
        {
            UdpClient udpClient = new UdpClient();
            udpClient.Connect(textBoxIP.Text, Int32.Parse(textBoxPort.Text));
            Byte[] senddata = Encoding.ASCII.GetBytes("On");
            udpClient.Send(senddata, senddata.Length);
        }

        private void buttonLedOff_Click(object sender, EventArgs e)
        {
            UdpClient udpClient = new UdpClient();
            udpClient.Connect(textBoxIP.Text, Int32.Parse(textBoxPort.Text));
            Byte[] senddata = Encoding.ASCII.GetBytes("Off");
            udpClient.Send(senddata, senddata.Length);
        }
        private void UDP_Send_Receive_FormClosing(Object sender, FormClosingEventArgs e)
        {
            thdUDPServer.Abort();
            Application.Exit();
        }

        private void groupBox1_Enter_1(object sender, EventArgs e)
        {

        }

        private void UDP_Send_Receive_Load(object sender, EventArgs e)
        {
            thdUDPServer = new Thread(new ThreadStart(serverThread));
            thdUDPServer.Start();
        }

            }
}


Program arduino

#include<ESP8266WiFi.h>
#include <WiFiUdp.h>

const char* ssid = "Andromax-M2Y-6637";
const char* password = "23813407";

WiFiUDP Udp;
unsigned int localUdpPort = 2000; // UDP Port yang dibuka pada sisi Wemos
char incomingPacket[255]; // Variabel untuk menampung data yang diterima, dalam bentuk byte array
char sendPacket[10]; // Variabel untuk mengirim data

unsigned long previousMillis = 0;
const long interval = 1000;

void setup()
{
pinMode(LED_BUILTIN, OUTPUT);
digitalWrite(LED_BUILTIN, 1);
Serial.begin(115200);
Serial.println();
// Koneksi ke WiFi
Serial.printf("Connecting to %s", ssid);
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED)
{
  delay(500);
Serial.print(".");
}
Serial.println(" connected");
// Membuka akses UDP Port
Udp.begin(localUdpPort);
Serial.printf("Now listening at IP %s, UDP port %d\n", WiFi.localIP().toString().c_str(),
localUdpPort);
}
void loop()
{
int packetSize = Udp.parsePacket();
// Jika ada data yang masuk
if (packetSize)
{
  Serial.printf("Received %d bytes from %s, port %d\n", packetSize,
Udp.remoteIP().toString().c_str(), Udp.remotePort());
int len = Udp.read(incomingPacket, 255);
if (len > 0)
{
incomingPacket[len] = 0;
}
Serial.printf("UDP packet contents: %s\n", incomingPacket);
// Convert byte array ke string
String data(incomingPacket);
// Cek data
if (data.equals("On")) {
digitalWrite(LED_BUILTIN, 0);
}
else if (data.equals("Off")) {
digitalWrite(LED_BUILTIN, 1);
}
}
unsigned long currentMillis = millis();
// Kirim data tiap 1 second
if (currentMillis - previousMillis >= interval) {
previousMillis = currentMillis;
// Generate data random
String tempVal = String(random(0, 100));
// Convert string ke byte array
tempVal.toCharArray(sendPacket, 10);
// Kirim data ke IP dan Port Tujuan
Udp.beginPacket("192.168.1.100", 2000);
Udp.write(sendPacket);
Udp.endPacket();
}
}

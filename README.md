```C#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Net;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading;
using System.Windows.Forms;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace SystemTrayClock
{
    public partial class SystemTrayClock : Form
    {
        [System.Runtime.InteropServices.DllImport("user32")]
        private static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
        [System.Runtime.InteropServices.DllImport("user32")]
        private static extern IntPtr FindWindowEx(IntPtr hwndParent, IntPtr hwndChildAfter, string className, string windowName);
        [System.Runtime.InteropServices.DllImport("user32")]
        private static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter, int X, int Y, int cx, int cy, uint uFlags);
        [System.Runtime.InteropServices.DllImport("user32")]
        private static extern IntPtr GetNextWindow(IntPtr hwnd, int wFlag);
        [System.Runtime.InteropServices.DllImport("user32")]
        private static extern bool GetWindowRect(IntPtr hwnd, out WindowRect rect);

        public struct WindowRect
        {
            public int left;
            public int top;
            public int right;
            public int bottom;
        }

        public SystemTrayClock()
        {
            InitializeComponent();
        }

        private void SystemTrayClock_Load(object sender, EventArgs e)
        {
            try
            {
                //取得系统托盘时间区域。
                IntPtr taskBarWnd = FindWindow("Shell_TrayWnd", null);
                IntPtr tray = FindWindowEx(taskBarWnd, IntPtr.Zero, "TrayNotifyWnd", null);
                IntPtr trayclock = FindWindowEx(tray, IntPtr.Zero, "TrayClockWClass", null);

                WindowRect rect = new WindowRect();
                GetWindowRect(trayclock, out rect);

                int trayclock_width = (Screen.PrimaryScreen.Bounds.Width - rect.left) - (Screen.PrimaryScreen.Bounds.Width - rect.right);
                int trayclock_height = (Screen.PrimaryScreen.Bounds.Height - rect.top) - (Screen.PrimaryScreen.Bounds.Height - rect.bottom);

                this.ShowInTaskbar = false;
                this.FormBorderStyle = System.Windows.Forms.FormBorderStyle.None;
                this.Height = trayclock_height;
                this.Width = trayclock_width;
                this.TopMost = true;
                int x = rect.left;
                int y = rect.top;
                this.Location = new Point(x, y);
                //屏蔽 显示桌面与win+D。
                Thread mustTop = new Thread(() =>
                {
                    while (!this.IsDisposed)
                    {
                        this.Invoke(new Action(() =>
                        {
                            this.TopMost = true;
                            this.lblTime.Text = DateTime.Now.ToString("HH:mm MM/dd");
                        }));
                    }
                });
                mustTop.Start();
            }
            catch (Exception ex)
            { 

            }

        }

        private void SystemTrayClock_FormClosing(object sender, FormClosingEventArgs e)
        {

        }

        private void getWeatherInfo()
        {
            string url = "http://t.weather.sojson.com/api/weather/city/101070201";
            string res = DoGetRequestSendData(url);
            JObject jo = (JObject)JsonConvert.DeserializeObject(res);
            string temperature = jo["data"]["wendu"].ToString();
            string weatherInfo = jo["data"]["forecast"][0]["fx"].ToString() + " " + jo["data"]["forecast"][0]["type"].ToString();
            this.lblWeather.Text = temperature + "℃" + " " + jo["data"]["forecast"][0]["type"].ToString();
        }

        private string DoGetRequestSendData(string url)
        {
            HttpWebRequest hwRequest = null;
            HttpWebResponse hwResponse = null;

            string strResult = string.Empty;
            try
            {
                hwRequest = (System.Net.HttpWebRequest)WebRequest.Create(url);
                //hwRequest.Timeout = 30000;
                hwRequest.Method = "GET";
                hwRequest.ContentType = "application/x-www-form-urlencoded";
            }
            catch (System.Exception err)
            {
                this.lblWeather.Text = "天气异常";
            }
            try
            {
                hwResponse = (HttpWebResponse)hwRequest.GetResponse();
                StreamReader srReader = new StreamReader(hwResponse.GetResponseStream(), Encoding.UTF8);
                strResult = srReader.ReadToEnd();
                srReader.Close();
                hwResponse.Close();
            }
            catch (System.Exception err)
            {
                this.lblWeather.Text = "天气异常";
            }
            return strResult;
        }

        private void 退出ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            //this.IsDisposed = false;
            System.Environment.Exit(0); 
        }

    }
}


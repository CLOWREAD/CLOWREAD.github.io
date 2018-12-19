---
layout:     post
title:      "UWP SerialPort" 
subtitle:   ""
date:       2018-12-12 15:05:00
author:     "SiyuanWang"
#header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - UWP 
    - SerialPort
---
# UWP SerialPort
## sample  

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Runtime.InteropServices.WindowsRuntime;
        using System.Text;
        using System.Threading.Tasks;
        using Windows.Devices.Enumeration;
        using Windows.Devices.SerialCommunication;
        using Windows.Storage.Streams;
        using Windows.Foundation;
        using System.Threading;
        using System.ComponentModel;

        namespace GasTestPrototype.SerialPort
        {
            public class SerialPortThread
            {


                

                public int Length
                {
                    get
                    {
                        return (m_Front - m_Rear+ m_DataPool.Length)% m_DataPool.Length;
                    }
                }


                private Byte[] m_DataPool = new Byte[1];
                private int m_Front =0, m_Rear=0;


                public SerialPortThread()
                {
                    m_DataPool = new Byte[4096];
                }
                public SerialPortThread( int bufferlen)
                {
                    m_DataPool = new Byte[bufferlen];
                }

            public  bool Put(Byte b)
                {
                    int bl = m_DataPool.Length;

                    if((m_Rear+1)%bl==m_Front)
                    {
                        return false;

                    }else
                    {
                        m_DataPool[m_Rear] = b;

                        m_Rear++;
                        m_Rear = m_Rear % bl;

                    }
                    return true;

                }
            public  Byte Get( out bool H)
                {
                    int bl = m_DataPool.Length;

                    Byte b=0;

                    if (m_Rear  == m_Front)
                    {
                        H = false;

                    }else
                    {
                        b= m_DataPool[m_Front] ;
                        m_Front++;
                        m_Front = m_Front % bl;
                        H = true;
                    }

                    return b;
                }

                public async void StartSerialThread(SerialDevice sd)
                {
                    m_Front = 0; m_Rear = 0;

                    if (sd==null)
                    {
                        return;
                    }
                    await Task.Run(async ()=>
                    {
                        Byte [] b=new Byte[1];
                        IBuffer buffer = b.AsBuffer();
                        try
                        {
                        while(true)
                            {


                            

                                await sd.InputStream.ReadAsync(buffer, buffer.Capacity, InputStreamOptions.None) ;

                                if (sd.BytesReceived > 0)
                                {
                                    Put(b[0]);
                                    //System.Diagnostics.Debug.WriteLine((char)b[0]);
                                }





                            }

                        }catch(Exception ex)
                        {
                            //ssd.Dispose();
                            System.Diagnostics.Debug.WriteLine("@@ Serial Thread Exit");
                            sd = null;
                        }

                    }

                        );

                }


            }
        }

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Runtime.InteropServices.WindowsRuntime;
        using System.Text;
        using System.Threading.Tasks;
        using Windows.Devices.Enumeration;
        using Windows.Devices.SerialCommunication;
        using Windows.Storage.Streams;
        using Windows.Foundation;
        using System.Threading;
        using System.ComponentModel;
        using Windows.UI.Xaml;

        namespace GasTestPrototype.SerialPort
        {
            class SerialPortUtil
            {
                public SerialDevice device;
                public DataReader reader;
                public DataWriter writer;
                public async Task OpenAsync()
                {
                    var aqsFilter = SerialDevice.GetDeviceSelector("COM3");
                    var devices = await DeviceInformation.FindAllAsync(aqsFilter);
                    if (devices.Any())
                    {

                    
                        for( int i=0;i< devices.Count;i++)
                        {
                            var deviceId = devices.ElementAt(i).Id;
                            this.device = await SerialDevice.FromIdAsync(deviceId);
                            if(this.device!=null)
                            {
                                break;
                            }
                        }

                        if (this.device != null)
                        {
                            this.device.BaudRate = 9600;
                            this.device.StopBits = SerialStopBitCount.One;
                            this.device.DataBits = 8;
                            this.device.Parity = SerialParity.None;
                            //
                            //this.device.Handshake = SerialHandshake.None;
                            //
                            this.device.WriteTimeout = new TimeSpan(0, 0, 3);
                            this.device.ReadTimeout = new TimeSpan(0, 0, 3);
                            //


                            this.reader = new DataReader(this.device.InputStream);
                            this.writer = new DataWriter(this.device.OutputStream);
                        }
                    }
                }

                public static async Task SendAsync(SerialDevice sd, String str)
                {
                    Byte[] b = new Byte[str.Length];
                    char[] c = str.ToCharArray();
                    for( int i=0;i<b.Length;i++)
                    {
                        b[i] = (Byte)c[i];
                    }
                    
                    await sd.OutputStream.WriteAsync(b.AsBuffer());
                }

                public static async Task<string> ReadAsync(SerialDevice sd, uint len,uint Timeout=10)

                {
                    String res = "";

                    await Task.Run(() =>
                    {
                    res=   Read( );
                    });

                    return res;
                }


                public static SerialPort.SerialPortThread g_SerialThread = new SerialPort.SerialPortThread();

                public static void StartReadThread(SerialDevice sd)
                {
                    SerialPortUtil.g_SerialThread.StartSerialThread(sd);
                }

                public static  String Read( uint Timeout = 10)
                {

                

                    bool Res;
                    Byte b;

                    bool startflag=false;
                    int QuitCount = 16;
                    StringBuilder resstr=new StringBuilder();


                    for (; ; )
                    {
                        b= g_SerialThread.Get(out Res);
                        if(Res==true)
                        {
                            startflag = true;
                            QuitCount = 16;
                            resstr.Append((char)b);
                        }
                        else
                        {

                            if (startflag==true)
                            {
                                QuitCount--;
                            }

                            if(QuitCount<=0)
                            {
                                break;
                            }
                            Task.Delay(100).Wait();
                        }



                    }

                    return resstr.ToString();
                }


            }
        }

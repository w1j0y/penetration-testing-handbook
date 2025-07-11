We can now use [](https://dotnetfiddle.net/)[https://dotnetfiddle.net](https://dotnetfiddle.net) to create a simple C# program online and decrypt user’s password.
```
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class Program
{
	public static void Main()
	{
		try
            {
				string password = DecryptString("BQO5l5Kj9MdErXx6Q6AGOw==", "c4scadek3y654321");
				Console.WriteLine("Decrypted Pwd: {0}", password);
            }
            catch (Exception e)
            {
                Console.WriteLine("Error: {0}", e.Message);
            }
		
	}
	
	public static string DecryptString(string EncryptedString, string Key)
	{
		byte[] array = Convert.FromBase64String(EncryptedString);
		Aes aes = Aes.Create();
		aes.KeySize = 128;
		aes.BlockSize = 128;
		aes.IV = Encoding.UTF8.GetBytes("1tdyjCbY1Ix49842");
		aes.Mode = CipherMode.CBC;
		aes.Key = Encoding.UTF8.GetBytes(Key);
		using (MemoryStream stream = new MemoryStream(array))
		{
			using (CryptoStream cryptoStream = new CryptoStream(stream, aes.CreateDecryptor(), CryptoStreamMode.Read))
			{
				byte[] array2 = new byte[checked(array.Length - 1 + 1)];
				cryptoStream.Read(array2, 0, array2.Length);
				return Encoding.UTF8.GetString(array2);
			}
		}
	}
}
```

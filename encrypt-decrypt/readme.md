#Lab: Encrypting and Decrypting Content  

##Introduction 
In this lab you will learn how to encrypt and decrypt content. The code examples are designed for a Windows Forms application. This application demonstrates the fundamentals of encryption and decryption. 

This walkthrough uses the following guidelines for encryption:

•	Use the RijndaelManaged class, a symmetric algorithm, to encrypt and decrypt data by using its automatically generated Key and IV.

•	Use the RSACryptoServiceProvider, an asymmetric algorithm, to encrypt and decrypt the key to the data encrypted by RijndaelManaged Asymmetric algorithms are best used for smaller amounts of data, such as a key.

##Objectives 
After completing this lab, you will be able to:

•	Use the RijndaelManaged class, a symmetric algorithm, to encrypt and decrypt data by using its automatically generated Key and IV.

•	Use the RSACryptoServiceProvider, an asymmetric algorithm, to encrypt and decrypt the key to the data encrypted by RijndaelManaged.  

##Prerequisites
You must have the Visual Studio 2013 (or above) installed on the machine.  

## Step by Step Walkthrough

####Step 1: Declaring Class Variables   

1.	Double click the Cryptography.sln located in \Begin\Cryptography.sln

2.	Add the following code to the Form1.cs class. Edit the string variables for your environment and preferences.

```cs
        // Declare CspParmeters and RsaCryptoServiceProvider
        // objects with global scope of your Form class.
        CspParameters cspp = new CspParameters();
        RSACryptoServiceProvider rsa;

        // Path variables for source, encryption, and
        // decryption folders. Must end with a backslash.
        const string EncrFolder = @"..\..\Output\Encrypt\";
        const string DecrFolder = @"..\..\Output\Decrypt\";
        const string SrcFolder = @"..\..\Output\Docs\";

        // Public key file
        const string PubKeyFile = 
			@"..\..\Output\Encrypt\rsaPublicKey.txt";

        // Key container name for
        // private/public key value pair.
        const string keyName = "Key01";
```        
 
####Step 2: Creating Asymmetric Key    

In this task you will add code that will create an asymmetric key that encrypts and decrypts the RijndaelManaged key. This key was used to encrypt the content and it displays the key container name on the label control.

1.	Add the following code as the Click event handler for the Create Keys button (buttonCreateAsmKeys_Click).

```cs
    // Stores a key pair in the key container
    cspp.KeyContainerName = keyName;
    rsa = new RSACryptoServiceProvider(cspp);
    rsa.PersistKeyInCsp = true;
    if (rsa.PublicOnly == true)
	
 	label1.Text = "Key: " + cspp.KeyContainerName 
	 	+ " - Public Only";
    	else
       label1.Text = "Key: " + cspp.KeyContainerName 
	 	+ " - Full Key Pair";
```

####Step 3: Encrypting a File

This task involves two methods: the event handler method for the Encrypt File button (buttonEncryptFile_Click) and the EncryptFile method. The first method displays a dialog box for selecting a file and passes the file name to the second method, which performs the encryption.
The encrypted content, key, and IV are all saved to one FileStream, which is referred to as the encryption package.

The EncryptFile method does the following:

•	Creates a RijndaelManaged symmetric algorithm to encrypt the content.

•	Creates an RSACryptoServiceProvider object to encrypt the RijndaelManaged key.

•	Uses a CryptoStream object to read and encrypt the FileStream of the source file, in blocks of bytes, into a destination FileStream object for the encrypted file.

•	Determines the lengths of the encrypted key and IV, and creates byte arrays of their length values.

•	Writes the Key, IV, and their length values to the encrypted package.

The encryption package uses the following format:

•	Key length, bytes 0 - 3
•	IV length, bytes 4 - 7
•	Encrypted key
•	IV
•	Cipher text

You can use the lengths of the key and IV to determine the starting points and lengths of all parts of the encryption package, which can then be used to decrypt the file.

1.	Add the following code as the Click event handler for the Encrypt File button (buttonEncryptFile_Click).

```cs
 if (rsa == null)
                MessageBox.Show("Key not set.");
            else
            {

           // Display a dialog box to select a file to encrypt.
                openFileDialog1.InitialDirectory = SrcFolder;
             if (openFileDialog1.ShowDialog() == DialogResult.OK)
                {
                    string fName = openFileDialog1.FileName;
                    if (fName != null)
                    {
                        FileInfo fInfo = new FileInfo(fName);
                        // Pass the file name without the path.
                        string name = fInfo.FullName;
                        EncryptFile(name);
                    }
                }
            }

2.	Add EcryptFile Method to Form.cs


// Create instance of Rijndael for
            // symetric encryption of the data.
            RijndaelManaged rjndl = new RijndaelManaged();
            rjndl.KeySize = 256;
            rjndl.BlockSize = 256;
            rjndl.Mode = CipherMode.CBC;
            ICryptoTransform transform = rjndl.CreateEncryptor();

            // Use RSACryptoServiceProvider to
            // enrypt the Rijndael key.
            // rsa is previously instantiated: 
            //    rsa = new RSACryptoServiceProvider(cspp);
            byte[] keyEncrypted = rsa.Encrypt(rjndl.Key, false);

            // Create byte arrays to contain
            // the length values of the key and IV.
            byte[] LenK = new byte[4];
            byte[] LenIV = new byte[4];

            int lKey = keyEncrypted.Length;
            LenK = BitConverter.GetBytes(lKey);
            int lIV = rjndl.IV.Length;
            LenIV = BitConverter.GetBytes(lIV);

            // Write the following to the FileStream
            // for the encrypted file (outFs):
            // - length of the key
            // - length of the IV
            // - ecrypted key
            // - the IV
            // - the encrypted cipher content

            int startFileName = inFile.LastIndexOf("\\") + 1;
            // Change the file's extension to ".enc"
            string outFile = EncrFolder + inFile.Substring(startFileName, inFile.LastIndexOf(".") - startFileName) + ".enc";

            using (FileStream outFs = new FileStream(outFile, FileMode.Create))
            {

                outFs.Write(LenK, 0, 4);
                outFs.Write(LenIV, 0, 4);
                outFs.Write(keyEncrypted, 0, lKey);
                outFs.Write(rjndl.IV, 0, lIV);

                // Now write the cipher text using
                // a CryptoStream for encrypting.
                using (CryptoStream outStreamEncrypted = new CryptoStream(outFs, transform, CryptoStreamMode.Write))
                {

                    // By encrypting a chunk at
                    // a time, you can save memory
                    // and accommodate large files.
                    int count = 0;
                    int offset = 0;

                    // blockSizeBytes can be any arbitrary size.
                    int blockSizeBytes = rjndl.BlockSize / 8;
                    byte[] data = new byte[blockSizeBytes];
                    int bytesRead = 0;

                    using (FileStream inFs = new FileStream(inFile, FileMode.Open))
                    {
                        do
                        {
                            count = inFs.Read(data, 0, blockSizeBytes);
                            offset += count;
                            outStreamEncrypted.Write(data, 0, count);
                            bytesRead += blockSizeBytes;
                        }
                        while (count > 0);
                        inFs.Close();
                    }
                    outStreamEncrypted.FlushFinalBlock();
                    outStreamEncrypted.Close();
                }
                outFs.Close();
            }
```
####Step 4: Decrypting a File

This task involves two methods, the event handler method for the Decrypt File button (buttonEncryptFile_Click), and the DecryptFile method. The first method displays a dialog box for selecting a file and passes its file name to the second method, which performs the decryption.

The Decrypt method does the following:

• Creates a RijndaelManaged symmetric algorithm to decrypt the content.

• Reads the first eight bytes of the FileStream of the encrypted package into byte arrays to obtain the lengths of the encrypted key and the IV.

• Extracts the key and IV from the encryption package into byte arrays.

• Creates an RSACryptoServiceProvider object to decrypt the RijndaelManaged key.

• Uses a CryptoStream object to read and decrypt the cipher text section of the FileStreamencryption package, in blocks of bytes, into the FileStream object for the decrypted file. When this is finished, the decryption is completed.


1. Add the following code as the Click event handler for the Decrypt File button.

```cs
if (rsa == null)
        MessageBox.Show("Key not set.");
    else
    {
        // Display a dialog box to select the encrypted file.
        openFileDialog2.InitialDirectory = EncrFolder;
        if (openFileDialog2.ShowDialog() == DialogResult.OK)
        {
            string fName = openFileDialog2.FileName;
            if (fName != null)
            {
                FileInfo fi = new FileInfo(fName);
                string name = fi.Name;
                DecryptFile(name);
            }
        }
    }

1.	Add the following DecryptFile Method to Form.cs

private void DecryptFile(string inFile)
{

    // Create instance of Rijndael for
    // symetric decryption of the data.
    RijndaelManaged rjndl = new RijndaelManaged();
    rjndl.KeySize = 256;
    rjndl.BlockSize = 256;
    rjndl.Mode = CipherMode.CBC;

    // Create byte arrays to get the length of
    // the encrypted key and IV.
    // These values were stored as 4 bytes each
    // at the beginning of the encrypted package.
    byte[] LenK = new byte[4];
    byte[] LenIV = new byte[4];

    // Consruct the file name for the decrypted file.
    string outFile = DecrFolder + inFile.Substring(0, inFile.LastIndexOf(".")) + ".txt";

    // Use FileStream objects to read the encrypted
    // file (inFs) and save the decrypted file (outFs).
    using (FileStream inFs = new FileStream(EncrFolder + inFile, FileMode.Open))
    {

        inFs.Seek(0, SeekOrigin.Begin);
        inFs.Seek(0, SeekOrigin.Begin);
        inFs.Read(LenK, 0, 3);
        inFs.Seek(4, SeekOrigin.Begin);
        inFs.Read(LenIV, 0, 3);

        // Convert the lengths to integer values.
        int lenK = BitConverter.ToInt32(LenK, 0);
        int lenIV = BitConverter.ToInt32(LenIV, 0);

        // Determine the start postition of
        // the ciphter text (startC)
        // and its length(lenC).
        int startC = lenK + lenIV + 8;
        int lenC = (int)inFs.Length - startC;

        // Create the byte arrays for
        // the encrypted Rijndael key,
        // the IV, and the cipher text.
        byte[] KeyEncrypted = new byte[lenK];
        byte[] IV = new byte[lenIV];

        // Extract the key and IV
        // starting from index 8
        // after the length values.
        inFs.Seek(8, SeekOrigin.Begin);
        inFs.Read(KeyEncrypted, 0, lenK);
        inFs.Seek(8 + lenK, SeekOrigin.Begin);
        inFs.Read(IV, 0, lenIV);
        Directory.CreateDirectory(DecrFolder);
        // Use RSACryptoServiceProvider
        // to decrypt the Rijndael key.
        byte[] KeyDecrypted = rsa.Decrypt(KeyEncrypted, false);

        // Decrypt the key.
        ICryptoTransform transform = rjndl.CreateDecryptor(KeyDecrypted, IV);

        // Decrypt the cipher text from
        // from the FileSteam of the encrypted
        // file (inFs) into the FileStream
        // for the decrypted file (outFs).
        using (FileStream outFs = new FileStream(outFile, FileMode.Create))
        {

            int count = 0;
            int offset = 0;

            // blockSizeBytes can be any arbitrary size.
            int blockSizeBytes = rjndl.BlockSize / 8;
            byte[] data = new byte[blockSizeBytes];


            // By decrypting a chunk a time,
            // you can save memory and
            // accommodate large files.

            // Start at the beginning
            // of the cipher text.
            inFs.Seek(startC, SeekOrigin.Begin);
            using (CryptoStream outStreamDecrypted = new CryptoStream(outFs, transform, CryptoStreamMode.Write))
            {
                do
                {
                    count = inFs.Read(data, 0, blockSizeBytes);
                    offset += count;
                    outStreamDecrypted.Write(data, 0, count);

                }
                while (count > 0);

                outStreamDecrypted.FlushFinalBlock();
                outStreamDecrypted.Close();
            }
            outFs.Close();
        }
        inFs.Close();
    }

}
```

####Step 5: Exporting a Public Key

This task saves the key created by the Create Keys button to a file. It exports only the public parameters.

This task simulates the scenario of Alice giving Bob her public key so that he can encrypt files for her. He and others who have that public key will not be able to decrypt them because they do not have the full key pair with private parameters.

1.	Add the following code as the Click event handler for the Export Public Key button (buttonExportPublicKey_Click).

```cs
    // Save the public key created by the RSA
    // to a file. Caution, persisting the
    // key to a file is a security risk.
    Directory.CreateDirectory(EncrFolder);
    StreamWriter sw = new StreamWriter(PubKeyFile, false);
    sw.Write(rsa.ToXmlString(false));
    sw.Close();
```

####Step 6: Importing a Public Key

This task loads the key with only public parameters, as created by the Export Public Key button, and sets it as the key container name.

This task simulates the scenario of Bob loading Alice's key with only public parameters so he can encrypt files for her.

1.	Add the following code as the Click event handler for the Import Public Key button (buttonImportPublicKey_Click).

```cs
	    StreamReader sr = new StreamReader(PubKeyFile);
            cspp.KeyContainerName = keyName;
            rsa = new RSACryptoServiceProvider(cspp);
            string keytxt = sr.ReadToEnd();
            rsa.FromXmlString(keytxt);
            rsa.PersistKeyInCsp = true;
            if (rsa.PublicOnly == true)
                label1.Text = "Key: " + cspp.KeyContainerName + " - Public Only";
            else
       		         label1.Text = "Key: " + cspp.KeyContainerName + " - Full Key Pair";
            sr.Close();

```

####Step 7: Importing a Public Key

This task sets the key container name to the name of the key created by using the Create Keysbutton. The key container will contain the full key pair with private parameters. This task simulates the scenario of Alice using her private key to decrypt files encrypted by Bob.

1.	Add the following code as the Click event handler for the Get Private Key button (buttonGetPrivateKey_Click).

cspp.KeyContainerName = keyName;

            rsa = new RSACryptoServiceProvider(cspp);
            rsa.PersistKeyInCsp = true;

            if (rsa.PublicOnly == true)
                label1.Text = "Key: " + cspp.KeyContainerName + " - Public Only";
            else
                label1.Text = "Key: " + cspp.KeyContainerName + " - Full Key Pair";





## Testing the Application

After you have built the application, perform the following testing scenarios.

NOTE:  Locate \Begin\Output folder and make sure it has following folders inside it. These are already created for you.

*	Encrypt 
*	Decrypt
*	Docs

###Create Keys, Encrypt and Decrypt content

1.	Run the application by pressing F5.

2.	Click the Create Keys button. The label displays the key name and shows that it is a full key pair.

3.	Click the Export Public Key button. Note that exporting the public key parameters does not change the current key. Examine the public key created at <LabSourceDirectory>\Cryptography\Begin\Output\Encrypt\rsaPublicKey.txt

4.	Click the Encrypt File button and select the file \Begin\Output\Docs\samplefile.txt. Also, examine the content of encrypted file saved in the folder \Begin\Output\Encrypt\samplefile.enc 

5.	Click the Decrypt File button and select the file just encrypted i-e samplefile.enc

Examine the file just decrypted. The decrypted file is located at \Begin\Output\Decrypt\samplefile.txt 

6.	Close the application and restart it to test retrieving persisted key containers in the next scenario.

###Encrypt using the Public key

This scenario demonstrates having only the public key to encrypt a file for another person. Typically, that person would give you only the public key and withhold the private key for decryption.

1.	Click the Import Public Key button. The label displays the key name and shows that it is public only.

2.	Click the Encrypt File button and select the file \Begin\Output\Docs\samplefile.txt 

3.	Click the Decrypt File button and select the file just encrypted i-e samplefile.enc. This will fail with an exception because you must have the private key to decrypt.

7.	Close the application and restart it to test persisted private key in the next scenario.

###Decrypt using the Private key

1.	Click the Get Private Key button. The label displays the key name and shows whether it is the full key pair.

2.	Click the Decrypt File button and select the file just encrypted i-e samplefile.enc. This will be successful because you have the full key pair to decrypt.


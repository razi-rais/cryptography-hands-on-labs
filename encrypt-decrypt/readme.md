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

####Step 1: Declaring Class Variables   

1.	Double click the Cryptography.sln located in \Begin\Cryptography.sln

2.	Add the following code to the Form1.cs class. Edit the string variables for your environment and preferences.

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
        
 
####Step 2: Creating Asymmetric Key    

In this task you will add code that will create an asymmetric key that encrypts and decrypts the RijndaelManaged key. This key was used to encrypt the content and it displays the key container name on the label control.

1.	Add the following code as the Click event handler for the Create Keys button (buttonCreateAsmKeys_Click).

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

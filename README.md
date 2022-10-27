Encrypt/Decrypt Files in VB.NET (Using Rijndael)

How to encrypt and decrypt files using Rijndael. ?

Introduction

This is a sample application that will encrypt and decrypt files in VB.NET using Rijndael Managed. I started this project because I had several files on my computer that I didn’t want accessible to anyone but me. Sure I could have downloaded a free encryption program off the net, but what fun would that be? So being the hobbyist that I am, I decided to create my own.
Background

![2022-10-27_101542](https://user-images.githubusercontent.com/72635460/198356184-c0b28953-0fec-4349-9dfa-006af363f303.png)

Here are some brief descriptions of the cryptographic concepts relevant to this application. I am going to keep things as simple and basic as possible. If you want further details there is a ton of information on the web. Also check out “.NET Encryption Simplified” by: wumpus1 right here at Code Project. These descriptions are based on how the concepts are used in this application.

    The Key:

    The password used to encrypt/decrypt data.
    The IV:

    Initialization Vector. This is used to encrypt the first portion of the data to be encrypted.
    Rijndael:

    The algorithm used for encryption/decryption. In this application, Rijndael is using a 256 bit key and a 128 bit IV.
    SHA512 Hashing:

    This takes a string (the password) and transforms it into a fixed size (512 bits) of “encrypted data”. The same string will always “hash” into the same 512 bits of data.

Using the code

In this section I will cover the following:

    Creating the Key
    Creating the IV
    Encryption and decryption
    Changing file extensions
    Putting it all together 

My explanations will be brief because my code is heavily commented. Before we begin, we will need the following Imports statements:

Imports System
Imports System.IO
Imports System.Security
Imports System.Security.Cryptography

Now let's declare our global variables:

'*************************
'** Global Variables
'*************************

Dim strFileToEncrypt As String
Dim strFileToDecrypt As String
Dim strOutputEncrypt As String
Dim strOutputDecrypt As String
Dim fsInput As System.IO.FileStream
Dim fsOutput As System.IO.FileStream

Creating the Key

Of course, the most secure Key would be a randomly generated Key. But I prefer to make up my own. The following code is a function that will create a 256 bit Hashed Key from the user’s password. Here is what happens in the function:

    The function receives a string (the password).
    Converts the string to an array.
    Converts the array to a byte.
    Uses SHA512 to hash the byte.
    Stores the first 256 bits of the hashed byte into a new byte (the key).
    Returns the key. 

For a more in-depth description, read the ‘comments' in the following code:

'*************************
'** Create A Key
'*************************

Private Function CreateKey(ByVal strPassword As String) As Byte()
    'Convert strPassword to an array and store in chrData.
    Dim chrData() As Char = strPassword.ToCharArray
    'Use intLength to get strPassword size.
    Dim intLength As Integer = chrData.GetUpperBound(0)
    'Declare bytDataToHash and make it the same size as chrData.
    Dim bytDataToHash(intLength) As Byte
    
    'Use For Next to convert and store chrData into bytDataToHash.
    For i As Integer = 0 To chrData.GetUpperBound(0)
        bytDataToHash(i) = CByte(Asc(chrData(i)))
    Next

    'Declare what hash to use.
    Dim SHA512 As New System.Security.Cryptography.SHA512Managed
    'Declare bytResult, Hash bytDataToHash and store it in bytResult.
    Dim bytResult As Byte() = SHA512.ComputeHash(bytDataToHash)
    'Declare bytKey(31).  It will hold 256 bits.
    Dim bytKey(31) As Byte
    
    'Use For Next to put a specific size (256 bits) of 
    'bytResult into bytKey. The 0 To 31 will put the first 256 bits
    'of 512 bits into bytKey.
    For i As Integer = 0 To 31
        bytKey(i) = bytResult(i)
    Next

    Return bytKey 'Return the key.
End Function

Here is an alternate example of creating a Key without using SHA512 hashing:Private Function CreateKey(ByVal strPassword As String) As Byte()
    Dim bytKey As Byte()
    Dim bytSalt As Byte() = System.Text.Encoding.ASCII.GetBytes("salt")
    Dim pdb As New PasswordDeriveBytes(strPassword, bytSalt)
        
    bytKey = pdb.GetBytes(32)

    Return bytKey 'Return the key.
End Function

Creating the IV

OK, so we used the first 256 bits of our hashed byte to create the key. We will use the next 128 bits of our hashed byte to create our IV. This way the key will be different from the IV. This function is almost identical to the previous one.

'*************************
'** Create An IV
'*************************

Private Function CreateIV(ByVal strPassword As String) As Byte()
    'Convert strPassword to an array and store in chrData.
    Dim chrData() As Char = strPassword.ToCharArray
    'Use intLength to get strPassword size.
    Dim intLength As Integer = chrData.GetUpperBound(0)
    'Declare bytDataToHash and make it the same size as chrData.
    Dim bytDataToHash(intLength) As Byte

    'Use For Next to convert and store chrData into bytDataToHash.
    For i As Integer = 0 To chrData.GetUpperBound(0)
        bytDataToHash(i) = CByte(Asc(chrData(i)))
    Next

    'Declare what hash to use.
    Dim SHA512 As New System.Security.Cryptography.SHA512Managed
    'Declare bytResult, Hash bytDataToHash and store it in bytResult.
    Dim bytResult As Byte() = SHA512.ComputeHash(bytDataToHash)
    'Declare bytIV(15).  It will hold 128 bits.
    Dim bytIV(15) As Byte

    'Use For Next to put a specific size (128 bits) of bytResult into bytIV.
    'The 0 To 30 for bytKey used the first 256 bits of the hashed password.
    'The 32 To 47 will put the next 128 bits into bytIV.
    For i As Integer = 32 To 47
        bytIV(i - 32) = bytResult(i)
    Next

    Return bytIV 'Return the IV.
End Function

Here is an alternate example of creating an IV without using SHA512 hashing:

Private Function CreateIV(ByVal strPassword As String) As Byte()
    Dim bytIV As Byte()
    Dim bytSalt As Byte() = System.Text.Encoding.ASCII.GetBytes("salt")
    Dim pdb As New PasswordDeriveBytes(strPassword, bytSalt)

    bytIV = pdb.GetBytes(16)

    Return bytIV 'Return the IV.
End Function

Changing File Extensions

Basically, what we are doing here is taking the path name of the file to encrypt/decrypt and adding or removing an “.encrypt” extension. This is not really necessary for encrypting files, but I think it looks cool.

When encrypting a file we would add an “.encrypt” extension as follows:

'Setup the open dialog.
OpenFileDialog.FileName = ""
OpenFileDialog.Title = "Choose a file to encrypt"
OpenFileDialog.InitialDirectory = "C:\"
OpenFileDialog.Filter = "All Files (*.*) | *.*"

'Find out if the user chose a file.
If OpenFileDialog.ShowDialog = DialogResult.OK Then
    strFileToEncrypt = OpenFileDialog.FileName
    txtFileToEncrypt.Text = strFileToEncrypt

    Dim iPosition As Integer = 0
    Dim i As Integer = 0

    'Get the position of the last "\" in the OpenFileDialog.FileName path.
    '-1 is when the character your searching for is not there.
    'IndexOf searches from left to right.
    While strFileToEncrypt.IndexOf("\"c, i) <> -1
        iPosition = strFileToEncrypt.IndexOf("\"c, i)
        i = iPosition + 1
    End While

    'Assign strOutputFile to the position after the last "\" in the path.
    'This position is the beginning of the file name.
    strOutputEncrypt = strFileToEncrypt.Substring(iPosition + 1)
    'Assign S the entire path, ending at the last "\".
    Dim S As String = strFileToEncrypt.Substring(0, iPosition + 1)
    'Replace the "." in the file extension with "_".
    strOutputEncrypt = strOutputEncrypt.Replace("."c, "_"c)
    'The final file name.  XXXXX.encrypt
    txtDestinationEncrypt.Text = S + strOutputEncrypt + ".encrypt"
    
    When decrypting a file we would remove the “.encrypt” extension as follows:
    
    'Setup the open dialog.
OpenFileDialog.FileName = ""
OpenFileDialog.Title = "Choose a file to decrypt"
OpenFileDialog.InitialDirectory = "C:\"
OpenFileDialog.Filter = "Encrypted Files (*.encrypt) | *.encrypt"

'Find out if the user chose a file.
If OpenFileDialog.ShowDialog = DialogResult.OK Then
    strFileToDecrypt = OpenFileDialog.FileName
    txtFileToDecrypt.Text = strFileToDecrypt
    Dim iPosition As Integer = 0
    Dim i As Integer = 0
    'Get the position of the last "\" in the OpenFileDialog.FileName path.
    '-1 is when the character your searching for is not there.
    'IndexOf searches from left to right.

    While strFileToDecrypt.IndexOf("\"c, i) <> -1
        iPosition = strFileToDecrypt.IndexOf("\"c, i)
        i = iPosition + 1
    End While

    'strOutputFile = the file path minus the last 8 characters (.encrypt)
    strOutputDecrypt = strFileToDecrypt.Substring(0, _
                                            strFileToDecrypt.Length - 8)
    'Assign S the entire path, ending at the last "\".
    Dim S As String = strFileToDecrypt.Substring(0, iPosition + 1)
    'Assign strOutputFile to the position after the last "\" in the path.
    strOutputDecrypt = strOutputDecrypt.Substring((iPosition + 1))
    'Replace "_" with "."
    txtDestinationDecrypt.Text = S + strOutputDecrypt.Replace("_"c, "."c)
    
    Keep in mind that the above code is geared towards my sample application.
Putting It All Together

OK, so this is where everything comes together with the “Encrypt” and “Decrypt” buttons. Basically what happens here is:

    Variables are declared for the Key and IV.
    The user’s password is passed to the CreateKey function.
    The user’s password is passed to the CreateIV function.
    The input path name, output path name, Key, IV, and CryptoAction are passed to the EncryptOrDecryptFile procedure. 

Encrypting would go as follows:

'Declare variables for the key and iv.
'The key needs to hold 256 bits and the iv 128 bits.
Dim bytKey As Byte()
Dim bytIV As Byte()
'Send the password to the CreateKey function.
bytKey = CreateKey(txtPassEncrypt.Text)
'Send the password to the CreateIV function.
bytIV = CreateIV(txtPassEncrypt.Text)
'Start the encryption.
EncryptOrDecryptFile(strFileToEncrypt, txtDestinationEncrypt.Text, _
                     bytKey, bytIV, CryptoAction.ActionEncrypt)
                     
Decrypting would go as follows:

'Declare variables for the key and iv.
'The key needs to hold 256 bits and the iv 128 bits.
Dim bytKey As Byte()
Dim bytIV As Byte()
'Send the password to the CreateKey function.
bytKey = CreateKey(txtPassDecrypt.Text)
'Send the password to the CreateIV function.
bytIV = CreateIV(txtPassDecrypt.Text)
'Start the decryption.
EncryptOrDecryptFile(strFileToDecrypt, txtDestinationDecrypt.Text, _
                     bytKey, bytIV, CryptoAction.ActionDecrypt)
                     
Points of Interest

This application uses the Rijndael algorithm to encrypt and decrypt files. You could also use Data Encryption Standard (DES) or Triple DES to do the same. All you would need to change is the Key size, IV size and the Crypto Service Provider. I hope that you can have some fun with this code. Questions and comments are appreciated.
                     

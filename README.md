#!/usr/bin/env python
# coding: utf-8
def key_generator(pwd):
    key_list=[]
    for x in pwd:
        key_list.append(str(ord(x)))
    key=''.join(key_list)
    key=int(key)
    return key

# Encryption function
def encrypt(text,key):
    cipher_text=""
    for i in range(len(text)):
        char=text[i]
        cipher_text+=chr((ord(char)+key-32)%90+32)
    return cipher_text

# Decryption function
def decrypt(cipher,key):
    plain_text=""
    for i in range(len(cipher)):
        char=cipher[i]
        plain_text+=chr((ord(char)-key-32)%90+32)
    return plain_text

# function for convert a string into binary (list)
def str_to_bin(s):
    return [bin(ord(x))[2:].zfill(8) for x in s]

#function for convert a binary (list) into string
def bin_to_str(b):
    return ''.join([chr(int(x,2)) for x in b])

# Function for hiding/embedding text message into an image
def encode():
    image=input("Enter path of the image: ")
    
    message=str(input("Enter your message: "))
    
    password=input("Enter password for encryption: ")
    key=key_generator(password)
    
    cipher=encrypt(message,key)
    
    binary=str_to_bin(cipher)  # binary is a list
    
    # re-arrange the binary message
    flag='11111111'
    binary.insert(len(binary),flag)
    
    # concatenate data of a list into a single string
    msg_to_be_embed=''
    for x in binary:
        msg_to_be_embed+=x
    
    bits_of_msg=len(binary)*8
    
    # embedding starts here
    import cv2
    img=cv2.imread(image)
    rows,cols,channel=img.shape
    
    pixels=rows*cols
    blue,green,red=cv2.split(img)
    
    if(pixels>=bits_of_msg):
        for i in range(rows):
            for j in range(cols):
                if(len(msg_to_be_embed)!=0):
                    if((msg_to_be_embed[0]=='1' and red[i][j]%2==0) or (msg_to_be_embed[0]=='0' and red[i][j]%2==1)):
                        if(red[i][j]==255):
                            red[i][j]-=1
                        else:
                            red[i][j]+=1
                msg_to_be_embed=msg_to_be_embed[1:]
                
        en_img=cv2.merge((blue, green, red))
        cv2.imwrite('encrypted_image.png',en_img)
        print()
        print("Encryption successful!!")
    else:
        print("Your message is too large")
        
    return

# Function for retriving message from the encrypted image
def decode():
    import cv2
    
    flag='11111111'
    image=input("Enter the encrypted image or image path: ")
    password=input("Enter password: ")
    
    key=key_generator(password)
    
    img=cv2.imread(image)
    rows,cols,channel=img.shape
    blue, green, red=cv2.split(img)

    # retriving of string
    info=""
    for i in range(rows):
        for j in range(cols):
            info=info + str(red[i][j] % 2)
            
    end_index=info.find(flag)
    info=info[0:end_index]   # info is the retrived string

    # convert the retrived string into a list of binary values
    binary=[]
    for i in range(0, len(info), 8):
        temp_data = info[i:i + 8]
        binary.insert(len(binary),temp_data)
        
    # convert binary(list) into string(cipher)
    str_of_binary=bin_to_str(binary)
    
    # decrypt the encrypted message
    message=decrypt(str_of_binary,key)
    print()
    print("Your message is: ")
    print(message)
    
    return


# Main Function
choice=int(input("""Do you want to Encrypt or Decrypt?
    Press 1 to ENCRYPT.
    Press 2 to DECRYPT.
    """))

if choice==1:
    encode()
elif choice==2:
    decode()
else:
    print("""Wrong Choice.
        Please restart and choose a correct options! """)

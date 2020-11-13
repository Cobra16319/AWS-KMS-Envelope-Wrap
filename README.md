# AWS-KMS-Envelope-Wrap
How to encrypt everything

'aws kms create-key --description "This key is used for envelope encryption"'


Output: { "KeyMetadata": { "AWSAccountId": "************", "KeyId":


1. Generate Data Key
Let’s generate Data key using CMK we generated earlier. It returns Data Key (Plaintext) and Encrypted Data key (CiphertextBlob).


aws kms generate-data-key --key-id xxxxxxx-xxxx-xxxx-xxxx-************ --key-spec AES_256 
Output: 
{ "CiphertextBlob": "************

2. Decode Base64 encoded Data Key
Keep note that Data Key and Encrypted Data key generated in previous step are Base64 encoded so we need to decode it first.


echo '"data_key_string"=' | base64 --decode > ~/plaintext_data_key.txt

3. Encrypt Data using Plaintext Data Key
We are encrypting actual data using Decoded plaintext data key using AES256 encryption.

echo "this is what I want to encrypt and practice with to make sure this demo works out and I can ENCRYPT Everything!" | openssl enc -e -aes256 -k fileb://Users/someuser/plaintext_data_key.txt > ~/encrypted_data.txt

4. Package Encrypted Data and Data key
We have now Encrypted Data and Encrypted Data Key which we can store together or separately on Data store. Make sure to store Encrypted Data key which will be required during decryption.


5. Remove Plaintext Data Key
We can remove Data key from system after Data encryption as it’s sensitive information and we don’t require it as we have stored Encrypted Data key so in future whenever required we can get back plaintext data key.

rm ~/plaintext_data_key.txt


6. Extract Data for Decryption
Let’s we want our encoded data back so first need to extract Encrypted Data key we stored earlier and then Decode it as it was also Base64 encoded.

echo '************somelongboringstring.....==' | base64 --decode > ~/encrypted_data_key.txt

7. Decrypt Encrypted Plaintext Data Key
Once we get back Encrypted Data Key, we need to call Decrypt API to get Plaintext Data Key.


aws kms decrypt --ciphertext-blob fileb:///Users/someuser/encrypted_data_key.txt 
Output: 
{ "KeyId": "arn:aws:kms:us-east-1:************:key/xxxxxxxx-xxxx-xxxx-xxxx-************", "Plaintext": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=", "EncryptionAlgorithm": "SYMMETRIC_DEFAULT" }

8. Decode Base64 encoded Plaintext Data Key
Again Decrypted Data Key we got is Base64 encoded so we need to decode it first.

echo 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=' | base64 --decode > ~/decrypted_plaintext_data_key.txt

9. Decrypt actual data using Plaintext Data Key
Take actual encrypted data and decrypt it using same AES256 algorithm and we got actual data back.



cat ~/encrypted_data.txt | openssl enc -d -aes256 -k fileb:///Users/someuser/decrypted_plaintext_data_key.txt 

Output: 


"this is what I want to encrypt and practice with to make sure this demo works out and I can ENCRYPT Everything!" 


10. Remove Plaintext Data Key
Cleanup Plaintext Data Key.
rm ~/decrypted_plaintext_data_key.txt



This is how AWS internally performs Data encryption for large datasets in S3, EBS, RDS, etc.. when data encryption is enabled. Know you know part of the AWS managed services

















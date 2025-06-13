# Secure IPCL / Secure UCL

## Prerequisites

- RandomKeyID : This is the ID of the random key which shall be used to store the random seed in case of a PRNG.
- MasterKeyID : This is the ID of the master key.
- MasterKeyMACJobId : CMAC JobID using the MasterKeyID.
- SessionKeyIdMac : This is the ID of the session key for MAC.
- SessionKeyIdCip : This is the ID of the session key for Cip.
- SessionKeyMACJobId : CMAC JobID using the SessionKeyIdMac.
- SessionKeyCipJobId : Cip JobID using the SessionKeyIdCip.
- RandomJobID : JobID using the RandomKeyID

## Design

```mermaid
sequenceDiagram
    participant UCL
    participant UCL_Adapter
    participant Csm
    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_Initialize
    UCL_Adapter->>Csm:Csm_KeyElementGet(MasterKeyId, CRYPTO_KE_MAC_KEY, &DummykeyPtr[0], DummykeyLengthPtr)
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK
    
    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_Random
    UCL_Adapter->>UCL_Adapter: RandomResultLenPtr=8
    UCL_Adapter->>UCL_Adapter: If PRNG
    UCL_Adapter->>UCL_Adapter: Entropyval = x
    UCL_Adapter->>Csm: Csm_RandomSeed(RandomKeyId, seed=Entropyval, seedlength)
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter:Endif
    UCL_Adapter->>Csm: Csm_RandomGenerate(RandomJobID, &RandomResultPtr, &RandomResultLenPtr)
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL: if(RandomResultLenPtr == 8) return UCL_E_OK else UCL_E_NOK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_SetSessionKey
    UCL_Adapter->>Csm: Csm_KeyElementSet(SessionKeyIdMac, CRYPTO_KE_MAC_KEY, &SessionKeyPtr[0], SessionKeyLengthPtr)
    UCL_Adapter->>Csm: Csm_KeyElementSet(SessionKeyIdCip, CRYPTO_KE_CIPHER_KEY, &SessionKeyPtr[0], SessionKeyLengthPtr)
    Csm->>UCL_Adapter: 
    UCL_Adapter->>Csm: Csm_KeySetValid(SessionKeyIdMac)
    UCL_Adapter->>Csm: Csm_KeySetValid(SessionKeyIdCip)
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CMAC_Start
    Csm->>Csm: Sync Job
    UCL_Adapter->>UCL_Adapter: if master key
    UCL_Adapter->>Csm: Csm_MacGenerate(MasterKeyMACJobId, CRYPTO_OPERATIONMODE_START, &DummyDataPtr[0], DummyDataLength, &DummyMacPtr[0], &DummyMacPtrLength[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: else
    UCL_Adapter->>Csm: Csm_MacGenerate(SessionKeyMACJobId, CRYPTO_OPERATIONMODE_START, &DummyDataPtr[0], DummyDataLength, &DummyMacPtr[0], &DummyMacPtrLength[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: endif
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CMAC_Process
    Csm->>Csm: Sync Job
    UCL_Adapter->>UCL_Adapter: if master key
    UCL_Adapter->>Csm: Csm_MacGenerate(MasterKeyMACJobId, CRYPTO_OPERATIONMODE_UPDATE, &pData[0], Size, &Result_16byte_array[0], &Resultlen_16bytes[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: else
    UCL_Adapter->>Csm: Csm_MacGenerate(SessionKeyMACJobId, CRYPTO_OPERATIONMODE_UPDATE, &pData[0], Size, &Result_16byte_array[0], &Resultlen_16bytes[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: endif
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CMAC_Done
    Csm->>Csm: Sync Job
    UCL_Adapter->>UCL_Adapter: if master key
    UCL_Adapter->>Csm: Csm_MacGenerate(MasterKeyMACJobId, CRYPTO_OPERATIONMODE_FINISH, &DummyDataPtr[0], DummyDataLength, &DummyMacPtr[0], &DummyMacPtrLength[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: else
    UCL_Adapter->>Csm: Csm_MacGenerate(SessionKeyMACJobId, CRYPTO_OPERATIONMODE_FINISH, &DummyDataPtr[0], DummyDataLength, &DummyMacPtr[0], &DummyMacPtrLength[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: endif
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CBC_Start
    Csm->>Csm: Sync Job
    UCL_Adapter->>Csm: Csm_KeyElementSet(SessionKeyIdCip, CRYPTO_KE_MAC_KEY, &IV[0], IVLength)
    UCL_Adapter->>Csm: Csm_Encrypt(SessionKeyCBCEncJobId, CRYPTO_OPERATIONMODE_START, &DummyDataPtr[0], DummyDataLength, &DummyMacPtr[0], &DummyMacPtrLength[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>Csm: Csm_Decrypt(SessionKeyCBCDecJobId, CRYPTO_OPERATIONMODE_START, &DummyDataPtr[0], DummyDataLength, &Result_16byte_array[0], &Resultlen_16bytes[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: endif
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CBC_Encrypt
    UCL_Adapter->>Csm: Csm_Encrypt(SessionKeyCBCEncJobId, CRYPTO_OPERATIONMODE_UPDATE, &Data2Enc[0], DataLen, &EncData[0], &EncDataLen[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CBC_Decrypt
    UCL_Adapter->>Csm: Csm_Decrypt(SessionKeyCBCEncJobId, CRYPTO_OPERATIONMODE_UPDATE, &Data2Dec[0], DataLen, &DecData[0], &DecDataLen[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK

    UCL->>UCL_Adapter: UclALCryptoCSM_Impl_IUclALCrypto_CBC_Done
    UCL_Adapter->>Csm: Csm_Encrypt(SessionKeyCBCEncJobId, CRYPTO_OPERATIONMODE_FINISH, &Data2Enc[0], DataLen, &EncData[0], &EncDataLen[0])
    UCL_Adapter->>Csm: Csm_Decrypt(SessionKeyCBCEncJobId, CRYPTO_OPERATIONMODE_FINISH, &Data2Dec[0], DataLen, &DecData[0], &DecDataLen[0])
    Csm->>UCL_Adapter: 
    UCL_Adapter->>UCL_Adapter: endif
    UCL_Adapter->>UCL: if(E_NOT_OK) return UCL_E_NOK else return UCL_E_OK
```

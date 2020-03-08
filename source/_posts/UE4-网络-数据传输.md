---
title: UE4TCP数据传输
date: 2020-03-08 16:38:28
tags:
categories:
 - UE4
 - 网络
img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/UE4/Ue4Log.png
---

记录一下UE4 Tcp传输数据的客户端代码
方便日后使用

# 头文件
#include "Networking.h"

// 添加 Networking 和 Sockets
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "Networking", "Sockets"});

# 用到的数据结构
```c++
UENUM(BlueprintType)
enum class FMsgType:uint8
{
	EROOM_ENTER,
	EROOM_EXIT,
	EPLAYER_READY,
	EPLAYER_CANCEL_READY,
	EGAME_START,
	EROOM_PLAY_DATA,
	EGAME_OVER
};

UENUM(BlueprintType)
enum class FMsgVersion :uint8
{
	VERSION0 = 0,
	VERSION1 = 1
};

USTRUCT(BlueprintType)
struct FProtoHead
{
	GENERATED_BODY()
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
	FMsgType MsgType;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    FMsgVersion MsgVer;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    int32 BodyLen;
};

USTRUCT(BlueprintType)
struct FProtoBody
{
	GENERATED_BODY()
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    bool Result;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    int32 MsgId;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    TArray<int32> DataArray;
};

USTRUCT(BlueprintType)
struct FProtoMsg
{
	GENERATED_BODY()
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    FProtoHead Header;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
    FProtoBody Body;
};
```

# socket管理
```c++
// h
UFUNCTION(BlueprintCallable, Category = "TCP Network")
bool CreateSocket(const FString IPStr, int32 Port);

FIPv4Address Ip;
FSocket* ClientFd;

// cpp
bool AGameNetwork::CreateSocket(const FString IPStr, int32 Port)
{
    // 将字符串ip转换为点分十进制ip
    FIPv4Address::Parse(IPStr, Ip);

    // 将十进制ip 转化成网络地址
    TSharedPtr<FInternetAddr> addr = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateInternetAddr();
    addr->SetIp(Ip.Value);
    addr->SetPort(Port);

    // 创建TCP Socket 文件描述符
    ClientFd = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateSocket(NAME_Stream, TEXT("TCP SOCKET"), false);

    if (ClientFd->Connect(*addr))
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

# 数据发送和接收

```c++
// h
UFUNCTION(BlueprintCallable, Category = "TCP Network")
bool SendMsg(FProtoMsg Msg);

UFUNCTION(BlueprintCallable, Category = "TCP Network")
bool RecvMsg();

// cpp
bool AGameNetwork::SendMsg(FProtoMsg Msg)
{
	int Len;
	int Send;
	uint8* data;
	data = Encode(Msg, &Len);
	
	if (ClientFd->Send(data, Len, Send))
	{
		return true;
	}
	else
	{
		return false;
	}
}

bool AGameNetwork::RecvMsg()
{
	if (ClientFd != nullptr)
	{
		uint8 Buffer[1024]{};
		uint32 Size = 0;
		int32 Read = 0;
		while (ClientFd->HasPendingData(Size))
		{
			ClientFd->Recv(Buffer, 1024, Read);
			return Decode(Buffer, Read);
		}
	}
	return false;
}
```

# 数据序列化
```c++
// h
uint8* Encode(FProtoMsg Msg, int32* Len);
const static int HEADER_SIZE = 6;

// cpp
uint8* AGameNetwork::Encode(FProtoMsg Msg, int32* Len)
{
	Msg.Header.BodyLen = Msg.Body.DataArray.Num() * Msg.Body.DataArray.GetTypeSize() + 5;

	*Len = Msg.Header.BodyLen + HEADER_SIZE;
	// HEADER
	auto data = new uint8_t[*Len];
    auto pdata = data;
	memset(data, 0, *Len);

	*pdata = (uint8)Msg.Header.MsgType;
	pdata++;

	*pdata = (uint8)Msg.Header.MsgVer;
	pdata++;

	*(uint32_t*)pdata = Msg.Header.BodyLen;
	pdata += 4;

	// BODY
	pdata++;
	*(uint32_t*)pdata = Msg.Body.MsgId;
	pdata += 4;
	
	memcpy(pdata, Msg.Body.DataArray.GetData(), Msg.Header.BodyLen - 5);

	return data;
}
```

# 数据反序列化
```c++
// h
bool Decode(uint8* data, int32 len);
bool ParseHeader(uint8** r_data, int32* reserved_len, int32* parse_len);
bool ParseBody(uint8** r_data, int32* reserved_len, int32* parse_len);

enum class ParseStatus
{
    PARSE_INIT,
    PARSE_HEAD,
    PARSE_BODY
};

TArray<uint8> ReservedData;
FProtoMsg* CurrentMsg;
ParseStatus ParseStatus = ParseStatus::PARSE_INIT;
TArray<FProtoMsg> MsgArray;


// cpp
bool AGameNetwork::Decode(uint8* data, int32 len)
{
	// 防止沾包 直接将受到的数据全部按顺序存储
	for (int32 i = 0; i < len; ++i)
	{
		ReservedData.Push(data[i]);
	}

	int32 r_len = ReservedData.Num();
	uint8* r_data = ReservedData.GetData();
	int32 parse_len = 0;

	while (r_len > 0)
	{
		switch (ParseStatus)
		{
		case ParseStatus::PARSE_INIT:
			CurrentMsg = new FProtoMsg();
			ParseStatus = ParseStatus::PARSE_HEAD;

		case ParseStatus::PARSE_HEAD:
			if (!ParseHeader(&r_data, &r_len, &parse_len))
			{
				return false;
			}
		case ParseStatus::PARSE_BODY:
			if (!ParseBody(&r_data, &r_len, &parse_len))
			{
				return false;
			}
			if (CurrentMsg != nullptr)
			{
				MsgArray.Push(*CurrentMsg);
			}
			CurrentMsg = nullptr;
		}
	}
	for (int32 i = 0; i < parse_len; ++i)
	{
		ReservedData.Pop();
	}
	return true;
}

bool AGameNetwork::ParseHeader(uint8** r_data, int32* reserved_len, int32* parse_len)
{
	if (*reserved_len < HEADER_SIZE)
	{
		return false;
	}

	uint8* data = *r_data;

	CurrentMsg->Header.MsgType = StaticCast<FMsgType>(*data);
	data++;

	CurrentMsg->Header.MsgVer = StaticCast<FMsgVersion>(*data);
	data++;

	CurrentMsg->Header.BodyLen = *(uint32*)data;

	*reserved_len -= HEADER_SIZE;
	*parse_len += HEADER_SIZE;
	*r_data += HEADER_SIZE;

	ParseStatus = ParseStatus::PARSE_BODY;
	return true;
}

bool AGameNetwork::ParseBody(uint8** r_data, int32* reserved_len, int32* parse_len)
{
	if (*reserved_len < CurrentMsg->Header.BodyLen)
	{
		return false;
	}

	uint8* data = *r_data;
	// current_msg->body = data;
	CurrentMsg->Body.Result = (*data) == 1;
	data++;
	CurrentMsg->Body.MsgId = *(int32*)data;
	data += 4;
	for (int i = 0; i < ((CurrentMsg->Header.BodyLen - 5) / sizeof(int32)); ++i)
	{
		CurrentMsg->Body.DataArray.Add(*(int32*)data);
		data += 4;
	}


	*reserved_len -= CurrentMsg->Header.BodyLen;
	*parse_len += CurrentMsg->Header.BodyLen;
	*r_data += CurrentMsg->Header.BodyLen;

	ParseStatus = ParseStatus::PARSE_INIT;
	return true;
}
```

# 数据获取
```c++
// h
UFUNCTION(BlueprintCallable, Category = "TCP Network")
    bool IsEmpty();

UFUNCTION(BlueprintCallable, Category = "TCP Network")
    FProtoMsg Front();

UFUNCTION(BlueprintCallable, Category = "TCP Network")
    void Pop();

UFUNCTION(BlueprintCallable, Category = "TCP Network")
    TArray<FProtoMsg> PopBackAllMsg();


// cpp
bool AGameNetwork::IsEmpty()
{
	return MsgArray.Num() == 0;
}

FProtoMsg AGameNetwork::Front()
{
	return MsgArray.Top();
}

void AGameNetwork::Pop()
{
	// MsgArray.Pop();
}

TArray<FProtoMsg> AGameNetwork::PopBackAllMsg()
{
	TArray<FProtoMsg> MsgArrayTemp = MsgArray;
	MsgArray.Empty();
	return MsgArrayTemp;
}
```

# 完整代码-继承自GameStateBase
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameStateBase.h"
#include "Networking.h"
#include "GameNetwork.generated.h"


UENUM(BlueprintType)
enum class FMsgType:uint8
{
	EROOM_ENTER,
	EROOM_EXIT,
	EPLAYER_READY,
	EPLAYER_CANCEL_READY,
	EGAME_START,
	EROOM_PLAY_DATA,
	EGAME_OVER
};

UENUM(BlueprintType)
enum class FMsgVersion :uint8
{
	VERSION0 = 0,
	VERSION1 = 1
};

USTRUCT(BlueprintType)
struct FProtoHead
{
	GENERATED_BODY()
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
	FMsgType MsgType;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		FMsgVersion MsgVer;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		int32 BodyLen;
};

USTRUCT(BlueprintType)
struct FProtoBody
{
	GENERATED_BODY()
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		bool Result;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		int32 MsgId;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		TArray<int32> DataArray;
};

USTRUCT(BlueprintType)
struct FProtoMsg
{
	GENERATED_BODY()
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		FProtoHead Header;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mantra")
		FProtoBody Body;

};

UCLASS()
class PACMAN_API AGameNetwork : public AGameStateBase
{
	GENERATED_BODY()
	
public:
	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		bool CreateSocket(const FString IPStr, int32 Port);

	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		bool SendMsg(FProtoMsg Msg);

	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		bool RecvMsg();

	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		bool IsEmpty();

	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		FProtoMsg Front();

	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		void Pop();

	UFUNCTION(BlueprintCallable, Category = "TCP Network")
		TArray<FProtoMsg> PopBackAllMsg();

	FIPv4Address Ip;

	FSocket* ClientFd;

public:


	enum class ParseStatus
	{
		PARSE_INIT,
		PARSE_HEAD,
		PARSE_BODY
	};

	bool Decode(uint8* data, int32 len);

	uint8* Encode(FProtoMsg Msg, int32* Len);

	const static int HEADER_SIZE = 6;

	TArray<uint8> ReservedData;

	FProtoMsg* CurrentMsg;

	ParseStatus ParseStatus = ParseStatus::PARSE_INIT;

	TArray<FProtoMsg> MsgArray;

	bool ParseHeader(uint8** r_data, int32* reserved_len, int32* parse_len);

	bool ParseBody(uint8** r_data, int32* reserved_len, int32* parse_len);
};
```

```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "GameNetwork.h"

bool AGameNetwork::CreateSocket(const FString IPStr, int32 Port)
{
	// 将字符串ip转换为点分十进制ip
	FIPv4Address::Parse(IPStr, Ip);

	// 将十进制ip 转化成网络地址
	TSharedPtr<FInternetAddr> addr = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateInternetAddr();
	addr->SetIp(Ip.Value);
	addr->SetPort(Port);

	// 创建TCP Socket 文件描述符
	ClientFd = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateSocket(NAME_Stream, TEXT("TCP SOCKET"), false);


	if (ClientFd->Connect(*addr))
	{
		return true;
	}
	else
	{
		return false;
	}
}

bool AGameNetwork::SendMsg(FProtoMsg Msg)
{
	int Len;
	int Send;
	uint8* data;
	data = Encode(Msg, &Len);
	
	if (ClientFd->Send(data, Len, Send))
	{
		return true;
	}
	else
	{
		return false;
	}
}

bool AGameNetwork::RecvMsg()
{
	if (ClientFd != nullptr)
	{
		uint8 Buffer[1024]{};
		uint32 Size = 0;
		int32 Read = 0;
		while (ClientFd->HasPendingData(Size))
		{
			ClientFd->Recv(Buffer, 1024, Read);
			return Decode(Buffer, Read);
		}
	}
	return false;
}

bool AGameNetwork::Decode(uint8* data, int32 len)
{
	// 防止沾包 直接将受到的数据全部按顺序存储
	for (int32 i = 0; i < len; ++i)
	{
		ReservedData.Push(data[i]);
	}

	int32 r_len = ReservedData.Num();
	uint8* r_data = ReservedData.GetData();
	int32 parse_len = 0;

	while (r_len > 0)
	{
		switch (ParseStatus)
		{
		case ParseStatus::PARSE_INIT:
			CurrentMsg = new FProtoMsg();
			ParseStatus = ParseStatus::PARSE_HEAD;

		case ParseStatus::PARSE_HEAD:
			if (!ParseHeader(&r_data, &r_len, &parse_len))
			{
				return false;
			}
		case ParseStatus::PARSE_BODY:
			if (!ParseBody(&r_data, &r_len, &parse_len))
			{
				return false;
			}
			if (CurrentMsg != nullptr)
			{
				MsgArray.Push(*CurrentMsg);
			}
			CurrentMsg = nullptr;
		}
	}
	for (int32 i = 0; i < parse_len; ++i)
	{
		ReservedData.Pop();
	}
	return true;
}

uint8* AGameNetwork::Encode(FProtoMsg Msg, int32* Len)
{
	Msg.Header.BodyLen = Msg.Body.DataArray.Num() * Msg.Body.DataArray.GetTypeSize() + 5;

	*Len = Msg.Header.BodyLen + HEADER_SIZE;
	// HEADER
	auto* data = new uint8_t[*Len];
	memset(data, 0, *Len);

	*data = (uint8)Msg.Header.MsgType;
	data++;

	*data = (uint8)Msg.Header.MsgVer;
	data++;

	*(uint32_t*)data = Msg.Header.BodyLen;
	data += 4;

	// BODY
	data++;
	*(uint32_t*)data = Msg.Body.MsgId;
	data += 4;
	
	memcpy(data, Msg.Body.DataArray.GetData(), Msg.Header.BodyLen - 5);

	// 回退增加的部分
	return data - HEADER_SIZE - 5;
}

bool AGameNetwork::ParseHeader(uint8** r_data, int32* reserved_len, int32* parse_len)
{
	if (*reserved_len < HEADER_SIZE)
	{
		return false;
	}

	uint8* data = *r_data;

	CurrentMsg->Header.MsgType = StaticCast<FMsgType>(*data);
	data++;

	CurrentMsg->Header.MsgVer = StaticCast<FMsgVersion>(*data);
	data++;

	CurrentMsg->Header.BodyLen = *(uint32*)data;

	*reserved_len -= HEADER_SIZE;
	*parse_len += HEADER_SIZE;
	*r_data += HEADER_SIZE;

	ParseStatus = ParseStatus::PARSE_BODY;
	return true;
}

bool AGameNetwork::ParseBody(uint8** r_data, int32* reserved_len, int32* parse_len)
{
	if (*reserved_len < CurrentMsg->Header.BodyLen)
	{
		return false;
	}

	uint8* data = *r_data;
	// current_msg->body = data;
	CurrentMsg->Body.Result = (*data) == 1;
	data++;
	CurrentMsg->Body.MsgId = *(int32*)data;
	data += 4;
	for (int i = 0; i < ((CurrentMsg->Header.BodyLen - 5) / sizeof(int32)); ++i)
	{
		CurrentMsg->Body.DataArray.Add(*(int32*)data);
		data += 4;
	}


	*reserved_len -= CurrentMsg->Header.BodyLen;
	*parse_len += CurrentMsg->Header.BodyLen;
	*r_data += CurrentMsg->Header.BodyLen;

	ParseStatus = ParseStatus::PARSE_INIT;
	return true;
}

bool AGameNetwork::IsEmpty()
{
	return MsgArray.Num() == 0;
}

FProtoMsg AGameNetwork::Front()
{
	return MsgArray.Top();
}

void AGameNetwork::Pop()
{
	// MsgArray.Pop();
}

TArray<FProtoMsg> AGameNetwork::PopBackAllMsg()
{
	TArray<FProtoMsg> MsgArrayTemp = MsgArray;
	MsgArray.Empty();
	return MsgArrayTemp;
}
```
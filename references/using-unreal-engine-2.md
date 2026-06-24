# Using Unreal Engine (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
// Copyright Epic Games, Inc. All Rights Reserved.

#include "DiscordSocialUnrealCharacter.h"
#include "Engine/LocalPlayer.h"
#include "Camera/CameraComponent.h"
#include "Components/CapsuleComponent.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "GameFramework/SpringArmComponent.h"
#include "GameFramework/Controller.h"
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"
#include "InputActionValue.h"

DEFINE_LOG_CATEGORY(LogTemplateCharacter);

#define APPLICATION_ID 1111111111111111111

void ADiscordSocialUnrealCharacter::BeginPlay() {
	auto PlayerController = Cast<APlayerController>(GetController());
	auto LocalPlayer = PlayerController->GetLocalPlayer();
	Discord = ULocalPlayer::GetSubsystem<UDiscordLocalPlayerSubsystem>(LocalPlayer);
	auto LogCallback = FDiscordClientLogCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnLogMessage);
	FScriptDelegate StatusChanged;
	StatusChanged.BindUFunction(this, "OnStatusChanged");
	Discord->Client->AddLogCallback(LogCallback, EDiscordLoggingSeverity::Info);
	Discord->OnStatusChanged.Add(StatusChanged);
}

void ADiscordSocialUnrealCharacter::DiscordConnect() {
	CodeVerifier = Discord->Client->CreateAuthorizationCodeVerifier();
	auto AuthArgs = NewObject<UDiscordAuthorizationArgs>();
	AuthArgs->Init();
	AuthArgs->SetClientId(APPLICATION_ID);
	AuthArgs->SetScopes(UDiscordClient::GetDefaultPresenceScopes());
	AuthArgs->SetCodeChallenge(CodeVerifier->Challenge());
	Discord->Client->Authorize(AuthArgs, FDiscordClientAuthorizationCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnAuthorizeCompleted));
}

void ADiscordSocialUnrealCharacter::OnLogMessage(FString Message, EDiscordLoggingSeverity Severity) {
	UE_LOG(LogTemplateCharacter, Log, TEXT("[%s] %s"), *UEnum::GetValueAsString(Severity), *Message);
}

void ADiscordSocialUnrealCharacter::OnStatusChanged(EDiscordClientStatus Status, EDiscordClientError Error, int32 ErrorDetail) {
	UE_LOG(LogTemplateCharacter, Log, TEXT("Connection status: %s"), *UEnum::GetValueAsString(Status));
	if (Status == EDiscordClientStatus::Ready) {
		UE_LOG(LogTemplateCharacter, Log, TEXT("Connected to Discord! Ready to go! You can now start using Discord features."));
		UE_LOG(LogTemplateCharacter, Log, TEXT("Number of Relationships: %d"), Discord->Client->GetRelationships().Num());

		UDiscordActivity activity;
		activity.SetDetails("In competitive");
		activity.SetState("Diamond II");
	}
	else if (Error != EDiscordClientError::None) {
		UE_LOG(LogTemplateCharacter, Log, TEXT("Connection error: %s (Detail: %d)"), *UEnum::GetValueAsString(Error), ErrorDetail);
	}
}

void ADiscordSocialUnrealCharacter::OnAuthorizeCompleted(UDiscordClientResult* Result, FString Code, FString RedirectUri) {
	if (!Result->Successful()) {
		UE_LOG(LogTemplateCharacter, Error, TEXT("Discord authorization failed: %s"), *Result->Error());
		return;
	}

	UE_LOG(LogTemplateCharacter, Log, TEXT("Authorization successful! Getting access token..."));
	Discord->Client->GetToken(APPLICATION_ID, Code, CodeVerifier->Verifier(), RedirectUri, FDiscordClientTokenExchangeCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnTokenExchange));
}

void ADiscordSocialUnrealCharacter::OnTokenExchange(UDiscordClientResult* Result, FString AccessToken, FString RefreshToken, EDiscordAuthorizationTokenType TokenType, int32 ExpiresIn, FString Scope) {
	if (!Result->Successful()) {
		UE_LOG(LogTemplateCharacter, Error, TEXT("Discord token exchange failed: %s"), *Result->Error());
	}

	Discord->Client->UpdateToken(TokenType, AccessToken, FDiscordClientUpdateTokenCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnTokenUpdated));
}

void ADiscordSocialUnrealCharacter::OnTokenUpdated(UDiscordClientResult* Result) {
	Discord->Client->Connect();
}

//////////////////////////////////////////////////////////////////////////
// ADiscordSocialUnrealCharacter

ADiscordSocialUnrealCharacter::ADiscordSocialUnrealCharacter()
{
	// Set size for collision capsule
	GetCapsuleComponent()->InitCapsuleSize(42.f, 96.0f);

	// Don't rotate when the controller rotates. Let that just affect the camera.
	bUseControllerRotationPitch = false;
	bUseControllerRotationYaw = false;
	bUseControllerRotationRoll = false;

	// Configure character movement
	GetCharacterMovement()->bOrientRotationToMovement = true; // Character moves in the direction of input...
	GetCharacterMovement()->RotationRate = FRotator(0.0f, 500.0f, 0.0f); // ...at this rotation rate

	// Note: For faster iteration times these variables, and many more, can be tweaked in the Character Blueprint
	// instead of recompiling to adjust them
	GetCharacterMovement()->JumpZVelocity = 700.f;
	GetCharacterMovement()->AirControl = 0.35f;
	GetCharacterMovement()->MaxWalkSpeed = 500.f;
	GetCharacterMovement()->MinAnalogWalkSpeed = 20.f;
	GetCharacterMovement()->BrakingDecelerationWalking = 2000.f;
	GetCharacterMovement()->BrakingDecelerationFalling = 1500.0f;

	// Create a camera boom (pulls in towards the player if there is a collision)
	CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
	CameraBoom->SetupAttachment(RootComponent);
	CameraBoom->TargetArmLength = 400.0f; // The camera follows at this distance behind the character
	CameraBoom->bUsePawnControlRotation = true; // Rotate the arm based on the controller

	// Create a follow camera
	FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
	FollowCamera->SetupAttachment(CameraBoom, USpringArmComponent::SocketName); // Attach the camera to the end of the boom and let the boom adjust to match the controller orientation
	FollowCamera->bUsePawnControlRotation = false; // Camera does not rotate relative to arm

	// Note: The skeletal mesh and anim blueprint references on the Mesh component (inherited from Character)
	// are set in the derived blueprint asset named ThirdPersonCharacter (to avoid direct content references in C++)
}

//////////////////////////////////////////////////////////////////////////
// Input

void ADiscordSocialUnrealCharacter::NotifyControllerChanged()
{
	Super::NotifyControllerChanged();

	// Add Input Mapping Context
	if (APlayerController* PlayerController = Cast<APlayerController>(Controller))
	{
		if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer()))
		{
			Subsystem->AddMappingContext(DefaultMappingContext, 0);
		}
	}
}

void ADiscordSocialUnrealCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	// Set up action bindings
	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent)) {

		// Jumping
		EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
		EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Completed, this, &ACharacter::StopJumping);

		// Moving
		EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &ADiscordSocialUnrealCharacter::Move);

		// Looking
		EnhancedInputComponent->BindAction(LookAction, ETriggerEvent::Triggered, this, &ADiscordSocialUnrealCharacter::Look);
	}
	else
	{
		UE_LOG(LogTemplateCharacter, Error, TEXT("'%s' Failed to find an Enhanced Input component! This template is built to use the Enhanced Input system. If you intend to use the legacy system, then you will need to update this C++ file."), *GetNameSafe(this));
	}
}

void ADiscordSocialUnrealCharacter::Move(const FInputActionValue& Value)
{
	// input is a Vector2D
	FVector2D MovementVector = Value.Get<FVector2D>();

	if (Controller != nullptr)
	{
		// find out which way is forward
		const FRotator Rotation = Controller->GetControlRotation();
		const FRotator YawRotation(0, Rotation.Yaw, 0);

		// get forward vector
		const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);

		// get right vector
		const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

		// add movement
		AddMovementInput(ForwardDirection, MovementVector.Y);
		AddMovementInput(RightDirection, MovementVector.X);
	}
}

void ADiscordSocialUnrealCharacter::Look(const FInputActionValue& Value)
{
	// input is a Vector2D
	FVector2D LookAxisVector = Value.Get<FVector2D>();

	if (Controller != nullptr)
	{
		// add yaw and pitch input to controller
		AddControllerYawInput(LookAxisVector.X);
		AddControllerPitchInput(LookAxisVector.Y);
	}
}

```

The full DiscordSocialUnrealCharacter.h

```
// Copyright Epic Games, Inc. All Rights Reserved.

#pragma once

#include "CoreMinimal.h"
#include "DiscordLocalPlayerSubsystem.h"
#include "GameFramework/Character.h"
#include "Logging/LogMacros.h"
#include "DiscordSocialUnrealCharacter.generated.h"

class USpringArmComponent;
class UCameraComponent;
class UInputMappingContext;
class UInputAction;
struct FInputActionValue;

DECLARE_LOG_CATEGORY_EXTERN(LogTemplateCharacter, Log, All);

UCLASS(config=Game)
class ADiscordSocialUnrealCharacter : public ACharacter
{
	GENERATED_BODY()

	/** Camera boom positioning the camera behind the character */
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "true"))
	USpringArmComponent* CameraBoom;

	/** Follow camera */
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "true"))
	UCameraComponent* FollowCamera;

	/** MappingContext */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	UInputMappingContext* DefaultMappingContext;

	/** Jump Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	UInputAction* JumpAction;

	/** Move Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	UInputAction* MoveAction;

	/** Look Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	UInputAction* LookAction;

public:
	ADiscordSocialUnrealCharacter();

	UFUNCTION(Exec)
	void DiscordConnect();

	UFUNCTION()
	void OnStatusChanged(EDiscordClientStatus Status, EDiscordClientError Error, int32 ErrorDetail);

	UPROPERTY()
	UDiscordAuthorizationCodeVerifier* CodeVerifier;

	UPROPERTY()
	UDiscordLocalPlayerSubsystem* Discord;

protected:

	/** Called for movement input */
	void Move(const FInputActionValue& Value);

	/** Called for looking input */
	void Look(const FInputActionValue& Value);

	void BeginPlay();
	void OnLogMessage(FString Message, EDiscordLoggingSeverity Severity);
	void OnAuthorizeCompleted(UDiscordClientResult* Result, FString Code, FString RedirectUri);
	void OnRichPresenceUpdated(UDiscordClientResult* Result);
	void OnTokenExchange(UDiscordClientResult* Result, FString AccessToken, FString RefreshToken, EDiscordAuthorizationTokenType TokenType, int32 ExpiresIn, FString Scope);
	void OnTokenUpdated(UDiscordClientResult* Result);

protected:

	virtual void NotifyControllerChanged() override;

	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

public:
	/** Returns CameraBoom subobject **/
	FORCEINLINE class USpringArmComponent* GetCameraBoom() const { return CameraBoom; }
	/** Returns FollowCamera subobject **/
	FORCEINLINE class UCameraComponent* GetFollowCamera() const { return FollowCamera; }
};

```

---

### Conclusion

 The Discord Social SDK is now integrated into your C++ application. Review the completed work:

### What You’ve Built

- Set up a Discord application with OAuth2
- Added SDK logging and status monitoring
- ✅ Implemented user authentication flow
- ✅ Retrieved Discord relationships data
- ✅ Added Rich Presence support

### Key Concepts Learned

- Initializing and configuring the Discord SDK
- Managing authentication and connections
- Using Discord social features
- Handling asynchronous callbacks
- Monitoring SDK status and events

---

### Next Steps

 The Discord Social SDK is set up with Unreal Engine and authenticated with Discord. You can now add more social features to your project with the SDK.

### Creating a Unified Friends List

Build one friends list that merges Discord relationships with game-specific friendships

### Setting Rich Presence

Extend your game's rich presence with richer details and game invite support

### Managing Game Invites

Let players invite friends into a game session or party. For help, join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

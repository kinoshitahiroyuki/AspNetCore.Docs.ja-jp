# <a name="upload-files-sample-app"></a><span data-ttu-id="1a191-101">ファイルのアップロードのサンプル アプリ</span><span class="sxs-lookup"><span data-stu-id="1a191-101">Upload Files Sample App</span></span>

<span data-ttu-id="1a191-102">このサンプル アプリでは、「[ASP.NET Core でのファイルのアップロード](https://docs.microsoft.com/aspnet/core/mvc/models/file-uploads)」トピックで説明されている概念を示します。</span><span class="sxs-lookup"><span data-stu-id="1a191-102">This sample app demonstrates concepts described in the [Upload files in ASP.NET Core](https://docs.microsoft.com/aspnet/core/mvc/models/file-uploads) topic.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="1a191-103">セキュリティの考慮事項</span><span class="sxs-lookup"><span data-stu-id="1a191-103">Security considerations</span></span>

<span data-ttu-id="1a191-104">サーバーにファイルをアップロードする機能をユーザーに提供するときは、十分に注意してください。</span><span class="sxs-lookup"><span data-stu-id="1a191-104">Use caution when providing users with the ability to upload files to a server.</span></span> <span data-ttu-id="1a191-105">攻撃者が、[サービス拒否](/windows-hardware/drivers/ifs/denial-of-service)攻撃を実行したり、ウイルスやマルウェアのアップロードを試みたり、他の方法でネットワークやサーバーを侵害しようとしたりする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="1a191-105">Attackers may execute [denial of service](/windows-hardware/drivers/ifs/denial-of-service) attacks, attempt to upload viruses or malware, or attempt to compromise networks and servers in other ways.</span></span>

<span data-ttu-id="1a191-106">攻撃の成功の可能性を少なくするセキュリティ手順は、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="1a191-106">Security steps that reduce the likelihood of a successful attack are:</span></span>

* <span data-ttu-id="1a191-107">システムの専用のファイル アップロード領域 (できれば、システム ドライブ以外) にファイルをアップロードします。</span><span class="sxs-lookup"><span data-stu-id="1a191-107">Upload files to a dedicated file upload area on the system, preferably to a non-system drive.</span></span> <span data-ttu-id="1a191-108">専用の場所を使用すると、アップロードされるファイルにセキュリティ対策を適用しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="1a191-108">Use of a dedicated location makes it easier to impose security measures on uploaded files.</span></span> <span data-ttu-id="1a191-109">ファイルのアップロード場所に対する実行アクセス許可を無効にします。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-109">Disable execute permissions on the file upload location.&dagger;</span></span>
* <span data-ttu-id="1a191-110">アプリと同じディレクトリ ツリーに、アップロードしたファイルを保持しないでください。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-110">Never persist uploaded files in the same directory tree as the app.&dagger;</span></span>
* <span data-ttu-id="1a191-111">アプリによって決められた安全なファイル名を使用します。</span><span class="sxs-lookup"><span data-stu-id="1a191-111">Use a safe file name determined by the app.</span></span> <span data-ttu-id="1a191-112">ユーザー入力によって指定されたファイル名や、アップロードされるファイルの信頼されていないファイル名は、使用しないでください。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-112">Don't use a file name provided by user input or the untrusted file name of the uploaded file.&dagger;</span></span>
* <span data-ttu-id="1a191-113">認められている特定のファイル拡張子のみを許可します。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-113">Only allow a specific set of approved file extensions.&dagger;</span></span>
* <span data-ttu-id="1a191-114">ユーザーが偽装ファイルをアップロードできないように (拡張子を *.txt* に変更して *.exe* ファイルをアップロードするなど)、ファイル形式のシグネチャを確認します。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-114">Check the file format signature to prevent a user from uploading a masqueraded file (for example, uploading an *.exe* file with a *.txt* extension).&dagger;</span></span>
* <span data-ttu-id="1a191-115">クライアント側のチェックがサーバーでも実行されることを確認します。</span><span class="sxs-lookup"><span data-stu-id="1a191-115">Verify that client-side checks are also performed on the server.</span></span> <span data-ttu-id="1a191-116">クライアント側のチェックは簡単に回避できます。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-116">Client-side checks are easy to circumvent.&dagger;</span></span>
* <span data-ttu-id="1a191-117">アップロードのサイズを確認し、アップロードのサイズが予想より大きくならないようにします。&dagger;</span><span class="sxs-lookup"><span data-stu-id="1a191-117">Check the size of the upload and prevent uploads that are larger than expected.&dagger;</span></span>
* <span data-ttu-id="1a191-118">同じ名前でアップロードされたファイルによってファイルが上書きされないようにする必要があるときは、ファイルをアップロードする前に、データベースまたは物理ストレージに対してファイル名を確認します。</span><span class="sxs-lookup"><span data-stu-id="1a191-118">When files shouldn't be overwritten by an uploaded file with the same name, check the file name against the database or physical storage before uploading the file.</span></span>
* <span data-ttu-id="1a191-119">**ファイルを格納する前に、アップロードされるコンテンツに対してウイルス/マルウェア スキャナーを実行します。**</span><span class="sxs-lookup"><span data-stu-id="1a191-119">**Run a virus/malware scanner on uploaded content before the file is stored.**</span></span>

<span data-ttu-id="1a191-120">&dagger;サンプル アプリでは、条件を満たす方法を示します。</span><span class="sxs-lookup"><span data-stu-id="1a191-120">&dagger;The sample app demonstrates an approach that meets the criteria.</span></span>

> [!WARNING]
> <span data-ttu-id="1a191-121">システムへの悪意のあるコードのアップロードは、頻繁に次のような内容のコードの実行するための足がかりとなります。</span><span class="sxs-lookup"><span data-stu-id="1a191-121">Uploading malicious code to a system is frequently the first step to executing code that can:</span></span>
>
> * <span data-ttu-id="1a191-122">システムを完全に乗っ取る。</span><span class="sxs-lookup"><span data-stu-id="1a191-122">Completely takeover a system.</span></span>
> * <span data-ttu-id="1a191-123">システムがクラッシュする結果で、システムを過負荷状態にする。</span><span class="sxs-lookup"><span data-stu-id="1a191-123">Overload a system with the result that the system crashes.</span></span>
> * <span data-ttu-id="1a191-124">ユーザーまたはシステムのデータを破壊する。</span><span class="sxs-lookup"><span data-stu-id="1a191-124">Compromise user or system data.</span></span>
> * <span data-ttu-id="1a191-125">パブリック UI に落書きする。</span><span class="sxs-lookup"><span data-stu-id="1a191-125">Apply graffiti to a public UI.</span></span>
>
> <span data-ttu-id="1a191-126">ユーザーからファイルを受け入れる際の外部アクセスによる攻撃を減らす方法については、次の資料を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1a191-126">For information on reducing the attack surface area when accepting files from users, see the following resources:</span></span>
>
> * [<span data-ttu-id="1a191-127">Unrestricted File Upload (ファイルの無制限のアップロード)</span><span class="sxs-lookup"><span data-stu-id="1a191-127">Unrestricted File Upload</span></span>](https://www.owasp.org/index.php/Unrestricted_File_Upload)
> * [<span data-ttu-id="1a191-128">Azure のセキュリティ: ユーザーからのファイルを受け入れるときは必ず適切な制御を行う</span><span class="sxs-lookup"><span data-stu-id="1a191-128">Azure Security: Ensure appropriate controls are in place when accepting files from users</span></span>](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

<span data-ttu-id="1a191-129">詳細については、「[ASP.NET Core でのファイルのアップロード](https://docs.microsoft.com/aspnet/core/mvc/models/file-uploads)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1a191-129">For additional information, see [Upload files in ASP.NET Core](https://docs.microsoft.com/aspnet/core/mvc/models/file-uploads).</span></span>

## <a name="how-to-use-the-sample"></a><span data-ttu-id="1a191-130">このサンプルを使用する方法</span><span class="sxs-lookup"><span data-stu-id="1a191-130">How to use the sample</span></span>

<span data-ttu-id="1a191-131">*appsettings.json* ファイルで、次のようにします。</span><span class="sxs-lookup"><span data-stu-id="1a191-131">In the *appsettings.json* file:</span></span>

1. <span data-ttu-id="1a191-132">格納されているファイルのパスを設定します (`StoredFilesPath`)。</span><span class="sxs-lookup"><span data-stu-id="1a191-132">Set the path for stored files (`StoredFilesPath`).</span></span>

   * <span data-ttu-id="1a191-133">サンプル アプリでは、値は `c:\\files` に設定されます。システムの C: ドライブのルートに、*files* という名前のフォルダーが存在することが想定されています。</span><span class="sxs-lookup"><span data-stu-id="1a191-133">The sample app sets the value to `c:\\files`, which assumes that a folder named *files* exists at the system's C: drive root.</span></span>
   * <span data-ttu-id="1a191-134">パスが存在している必要があります。</span><span class="sxs-lookup"><span data-stu-id="1a191-134">The path must exist.</span></span> <span data-ttu-id="1a191-135">システムの C: ドライブに *files* フォルダーを作成するか、パスを適切な場所に設定します。</span><span class="sxs-lookup"><span data-stu-id="1a191-135">Create a *files* folder on the system's C: drive or set the path to a suitable location.</span></span>
   * <span data-ttu-id="1a191-136">アプリのプロセスには、そのパスに対する読み取り/書き込みアクセス許可が必要です。</span><span class="sxs-lookup"><span data-stu-id="1a191-136">The app's process requires read/write permissions to the path.</span></span>
   * <span data-ttu-id="1a191-137">**重要!**</span><span class="sxs-lookup"><span data-stu-id="1a191-137">**IMPORTANT!**</span></span> <span data-ttu-id="1a191-138">そのパスですべてのユーザーに対して実行アクセス許可を無効にします。</span><span class="sxs-lookup"><span data-stu-id="1a191-138">Disable execute permissions for all users at the path.</span></span>

1. <span data-ttu-id="1a191-139">ファイル サイズの上限 (`FileSizeLimit`) をバイト単位で設定します。</span><span class="sxs-lookup"><span data-stu-id="1a191-139">Set the file size limit (`FileSizeLimit`) in bytes.</span></span> <span data-ttu-id="1a191-140">サンプル アプリの既定値 `2097152` (2,097,152 バイト) では、最大 2 MB のファイルをアップロードできます。</span><span class="sxs-lookup"><span data-stu-id="1a191-140">The sample app's default value of `2097152` (2,097,152 bytes) permits file uploads up to 2 MB.</span></span>
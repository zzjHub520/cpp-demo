# [C++ IPC类代码示例](https://vimsky.com/examples/detail/cpp-ex---IPC---class.html)

本文整理汇总了C++中**IPC类**的典型用法代码示例。如果您正苦于以下问题：C++ IPC类的具体用法？C++ IPC怎么用？C++ IPC使用的例子？那么恭喜您, 这里精选的类代码示例或许可以为您提供帮助。

在下文中一共展示了**IPC类**的15个代码示例，这些例子默认根据受欢迎程度排序。您可以为喜欢或者感觉有用的代码点赞，您的评价将有助于我们的系统推荐出更棒的C++代码示例。

## 示例1: Init

▲ 点赞 9 ▼

```cpp
bool
nsFileInputStream::Read(const IPC::Message *aMsg, void **aIter)
{
    using IPC::ReadParam;

    nsCString path;
    bool followLinks;
    PRInt32 flags;
    if (!ReadParam(aMsg, aIter, &path) ||
        !ReadParam(aMsg, aIter, &followLinks) ||
        !ReadParam(aMsg, aIter, &flags))
        return false;

    nsCOMPtr<nsIFile> file;
    nsresult rv = NS_NewNativeLocalFile(path, followLinks, getter_AddRefs(file));
    if (NS_FAILED(rv))
        return false;

    // IO flags = -1 means readonly, and
    // permissions are unimportant since we're reading
    rv = Init(file, -1, -1, flags);
    if (NS_FAILED(rv))
        return false;

    return true;
}
```

开发者ID:lofter2011，项目名称:Icefox，代码行数:26，代码来源:[nsFileStreams.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flofter2011%2FIcefox)





## 示例2: stream

▲ 点赞 7 ▼

```cpp
PRBool
nsMIMEInputStream::Read(const IPC::Message *aMsg, void **aIter)
{
    using IPC::ReadParam;

    if (!ReadParam(aMsg, aIter, &mHeaders) ||
        !ReadParam(aMsg, aIter, &mContentLength) ||
        !ReadParam(aMsg, aIter, &mStartedReading))
        return PR_FALSE;

    // nsMIMEInputStream::Init() already appended mHeaderStream & mCLStream
    mHeaderStream->ShareData(mHeaders.get(),
                             mStartedReading? mHeaders.Length() : 0);
    mCLStream->ShareData(mContentLength.get(),
                         mStartedReading? mContentLength.Length() : 0);

    IPC::InputStream inputStream;
    if (!ReadParam(aMsg, aIter, &inputStream))
        return PR_FALSE;

    nsCOMPtr<nsIInputStream> stream(inputStream);
    mData = stream;
    if (stream) {
        nsresult rv = mStream->AppendStream(mData);
        if (NS_FAILED(rv))
            return PR_FALSE;
    }

    if (!ReadParam(aMsg, aIter, &mAddContentLength))
        return PR_FALSE;

    return PR_TRUE;
}
```

开发者ID:Akin-Net，项目名称:mozilla-central，代码行数:33，代码来源:[nsMIMEInputStream.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FAkin-Net%2Fmozilla-central)



## 示例3: __declspec

▲ 点赞 5 ▼

```cpp
extern "C" __declspec(dllexport) bool ProcessStr(BSTR in_str, BSTR *out_str)
{
    IPC ipc;
    if (!ipc.isInit())
        return false;
    if (!ipc.process(in_str, out_str))
        return false;
    return true;
}
```

开发者ID:BackupTheBerlios，项目名称:sim-im-svn，代码行数:9，代码来源:[SimControl.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FBackupTheBerlios%2Fsim-im-svn)





## 示例4: WriteParam

▲ 点赞 2 ▼

```cpp
void
nsBufferedInputStream::Write(IPC::Message *aMsg)
{
    using IPC::WriteParam;

    WriteParam(aMsg, mBufferSize);

    IPC::InputStream inputStream(Source());
    WriteParam(aMsg, inputStream);
}
```

开发者ID:FunkyVerb，项目名称:devtools-window，代码行数:10，代码来源:[nsBufferedStreams.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FFunkyVerb%2Fdevtools-window)



## 示例5: ToString

▲ 点赞 1 ▼

```cpp
bool EditorMode::PlayProject(String addArgs, bool debug)
{
    ToolEnvironment* env = GetSubsystem<ToolEnvironment>();
    ToolSystem* tsystem = GetSubsystem<ToolSystem>();

    const String& editorBinary = env->GetEditorBinary();

    Project* project = tsystem->GetProject();

    if (!project)
        return false;

    Vector<String> paths;
    paths.Push(env->GetCoreDataDir());
    paths.Push(env->GetPlayerDataDir());
    paths.Push(project->GetResourcePath());

    // fixme: this is for loading from cache
    paths.Push(project->GetProjectPath());
    paths.Push(project->GetProjectPath() + "Cache");

    String resourcePaths;
    resourcePaths.Join(paths, "!");

    Vector<String> vargs;

    String args = ToString("--player --project \"%s\"", AddTrailingSlash(project->GetProjectPath()).CString());

    vargs = args.Split(' ');

    if (debug)
        vargs.Insert(0, "--debug");

    if (addArgs.Length() > 0)
        vargs.Insert(0, addArgs.Split(' '));

    String dump;
    dump.Join(vargs, " ");
    LOGINFOF("Launching Broker %s %s", editorBinary.CString(), dump.CString());

    IPC* ipc = GetSubsystem<IPC>();
    playerBroker_ = ipc->SpawnWorker(editorBinary, vargs);

    if (playerBroker_)
    {
        SubscribeToEvent(playerBroker_, E_IPCJSERROR, HANDLER(EditorMode, HandleIPCJSError));
        SubscribeToEvent(playerBroker_, E_IPCWORKEREXIT, HANDLER(EditorMode, HandleIPCWorkerExit));
        SubscribeToEvent(playerBroker_, E_IPCWORKERLOG, HANDLER(EditorMode, HandleIPCWorkerLog));
    }

    return playerBroker_.NotNull();

}
```

开发者ID:GREYFOXRGR，项目名称:AtomicGameEngine，代码行数:53，代码来源:[AEEditorMode.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FGREYFOXRGR%2FAtomicGameEngine)





## 示例6: WriteParam

▲ 点赞 1 ▼

```cpp
void
nsFileInputStream::Write(IPC::Message *aMsg)
{
    using IPC::WriteParam;

    nsCString path;
    mFile->GetNativePath(path);
    WriteParam(aMsg, path);
    bool followLinks;
    mFile->GetFollowLinks(&followLinks);
    WriteParam(aMsg, followLinks);
    WriteParam(aMsg, mBehaviorFlags);
}
```

开发者ID:lofter2011，项目名称:Icefox，代码行数:13，代码来源:[nsFileStreams.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flofter2011%2FIcefox)



## 示例7: WriteParam

▲ 点赞 1 ▼

```cpp
void
nsMIMEInputStream::Write(IPC::Message *aMsg)
{
    using IPC::WriteParam;

    WriteParam(aMsg, mHeaders);
    WriteParam(aMsg, mContentLength);
    WriteParam(aMsg, mStartedReading);

    IPC::InputStream inputStream(mData);
    WriteParam(aMsg, inputStream);

    WriteParam(aMsg, mAddContentLength);
}
```

开发者ID:Akin-Net，项目名称:mozilla-central，代码行数:14，代码来源:[nsMIMEInputStream.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FAkin-Net%2Fmozilla-central)





## 示例8: stream

▲ 点赞 1 ▼

```cpp
bool
nsBufferedInputStream::Read(const IPC::Message *aMsg, void **aIter)
{
    using IPC::ReadParam;

    PRUint32 bufferSize;
    IPC::InputStream inputStream;
    if (!ReadParam(aMsg, aIter, &bufferSize) ||
        !ReadParam(aMsg, aIter, &inputStream))
        return false;

    nsCOMPtr<nsIInputStream> stream(inputStream);
    nsresult rv = Init(stream, bufferSize);
    if (NS_FAILED(rv))
        return false;

    return true;
}
```

开发者ID:FunkyVerb，项目名称:devtools-window，代码行数:18，代码来源:[nsBufferedStreams.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FFunkyVerb%2Fdevtools-window)



## 示例9: GetData

▲ 点赞 1 ▼

```cpp
void
nsStringInputStream::Write(IPC::Message *aMsg)
{
#ifdef MOZ_IPC
    using IPC::WriteParam;

    nsCAutoString value;
    GetData(value);

    WriteParam(aMsg, value);
#endif
}
```

开发者ID:LittleForker，项目名称:mozilla-central，代码行数:12，代码来源:[nsStringStream.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FLittleForker%2Fmozilla-central)



## 示例10: stream

▲ 点赞 1 ▼

```cpp
PRBool
nsBufferedInputStream::Read(const IPC::Message *aMsg, void **aIter)
{
#ifdef MOZ_IPC
    using IPC::ReadParam;

    PRUint32 bufferSize;
    IPC::InputStream inputStream;
    if (!ReadParam(aMsg, aIter, &bufferSize) ||
        !ReadParam(aMsg, aIter, &inputStream))
        return PR_FALSE;

    nsCOMPtr<nsIInputStream> stream(inputStream);
    nsresult rv = Init(stream, bufferSize);
    if (NS_FAILED(rv))
        return PR_FALSE;

    return PR_TRUE;
#else
    return PR_FALSE;
#endif
}
```

开发者ID:mmmulani，项目名称:v8monkey，代码行数:22，代码来源:[nsBufferedStreams.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fmmmulani%2Fv8monkey)



## 示例11: while

▲ 点赞 1 ▼

```cpp
void Reprojector::load(IPC& ipc, bool flipped)
{
    bool serial_first_non_zero = false;
    string serial = "";

    for (int i = 4; i < 10; ++i)
    {
        string str_temp = "";
        str_temp += serial_number[i];

        if (!serial_first_non_zero && str_temp != "0")
            serial_first_non_zero = true;

        if (serial_first_non_zero)
            serial += serial_number[i];
    }

    bool has_complete_calib_data = false;

    if (directory_exists(data_path_current_module))
        if (file_exists(data_path_current_module + "\\0.jpg"))
            if (file_exists(data_path_current_module + "\\1.jpg"))
                if (file_exists(data_path_current_module + "\\stereoCalibData.txt"))
                    if (file_exists(data_path_current_module + "\\rect0.txt"))
                        if (file_exists(data_path_current_module + "\\rect1.txt"))
                            has_complete_calib_data = true;

    if (!has_complete_calib_data)
    {
        static bool block_thread = true;
        ipc.send_message("menu_plus", "show window", "");
        ipc.get_response("menu_plus", "show download", "", [](const string message_body)
        {
            COUT << "unblock" << endl;
            block_thread = false;
        });
        
        while (block_thread)
        {
            ipc.update();
            Sleep(100);
        }

        create_directory(data_path);
        create_directory(data_path_current_module);

        copy_file(executable_path + "\\rectifier.exe", data_path_current_module + "\\rectifier.exe");
        copy_file(executable_path + "\\opencv_core249.dll", data_path_current_module + "\\opencv_core249.dll");
        copy_file(executable_path + "\\opencv_highgui249.dll", data_path_current_module + "\\opencv_highgui249.dll");
        copy_file(executable_path + "\\opencv_imgproc249.dll", data_path_current_module + "\\opencv_imgproc249.dll");
        copy_file(executable_path + "\\opencv_calib3d249.dll", data_path_current_module + "\\opencv_calib3d249.dll");
        copy_file(executable_path + "\\opencv_flann249.dll", data_path_current_module + "\\opencv_flann249.dll");
        copy_file(executable_path + "\\opencv_features2d249.dll", data_path_current_module + "\\opencv_features2d249.dll");

        //http://s3-us-west-2.amazonaws.com/ractiv.com/data/
        //http://d2i9bzz66ghms6.cloudfront.net/data/

        string param0 = "http://s3-us-west-2.amazonaws.com/ractiv.com/data/" + serial + "/0.jpg";
        string param1 = data_path_current_module + "\\0.jpg";

        string* serial_ptr = &serial;
        IPC* ipc_ptr = &ipc;
        ipc.get_response("menu_plus", "download", param0 + "`" + param1, [serial_ptr, ipc_ptr](const string message_body)
        {
            if (message_body == "false")
                ipc_ptr->send_message("daemon_plus", "exit", "");
            else
            {
                string param0 = "http://s3-us-west-2.amazonaws.com/ractiv.com/data/" + *serial_ptr + "/1.jpg";
                string param1 = data_path_current_module + "\\1.jpg";

                ipc_ptr->get_response("menu_plus", "download", param0 + "`" + param1, [serial_ptr, ipc_ptr](const string message_body)
                {
                    if (message_body == "false")
                        ipc_ptr->send_message("daemon_plus", "exit", "");
                    else
                    {
                        string param0 = "http://s3-us-west-2.amazonaws.com/ractiv.com/data/" + *serial_ptr + "/stereoCalibData.txt";
                        string param1 = data_path_current_module + "\\stereoCalibData.txt";

                        ipc_ptr->get_response("menu_plus", "download", param0 + "`" + param1, [serial_ptr, ipc_ptr](const string message_body)
                        {
                            if (message_body == "false")
                                ipc_ptr->send_message("daemon_plus", "exit", "");
                            else
                            {
                                bool has_complete_calib_data = false;
                                while (!has_complete_calib_data)
                                {
                                    system(("cd " + cmd_quote + data_path_current_module + cmd_quote + " && rectifier.exe").c_str());

                                    if (directory_exists(data_path_current_module))
                                        if (file_exists(data_path_current_module + "\\0.jpg"))
                                            if (file_exists(data_path_current_module + "\\1.jpg"))
                                                if (file_exists(data_path_current_module + "\\stereoCalibData.txt"))
                                                    if (file_exists(data_path_current_module + "\\rect0.txt"))
                                                        if (file_exists(data_path_current_module + "\\rect1.txt"))
                                                            has_complete_calib_data = true;
                                }

//.........这里部分代码省略.........
```

开发者ID:nRoof，项目名称:touch_plus_source_code，代码行数:101，代码来源:[reprojector.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FnRoof%2Ftouch_plus_source_code)



## 示例12: main

▲ 点赞 1 ▼

```cpp
int main()
#endif

{
    // freopen("C:\\touch_plus_software_log.txt", "w", stdout);
    console_log_ipc = &ipc;	
    
#ifdef _WIN32
    char buffer[MAX_PATH];
    GetModuleFileName(NULL, buffer, MAX_PATH);
    string::size_type pos = string(buffer).find_last_of("\\/");
    executable_path = string(buffer).substr(0, pos);

#elif __APPLE__
    char path_buffer[1024];
    uint32_t path_size = sizeof(path_buffer);
    _NSGetExecutablePath(path_buffer, &path_size);
    string path_str(path_buffer);
    executable_path = path_str.substr(0, path_str.find_last_of("/"));
#endif

    data_path = executable_path + slash + "userdata";
    settings_file_path = data_path + slash + "settings.nrocinunerrad";
    ipc_path = executable_path + slash + "ipc";

    if (!directory_exists(ipc_path))
        create_directory(ipc_path);
    else
        delete_all_files(ipc_path);

    if (file_exists(executable_path + "/lock"))
        delete_file(executable_path + "/lock");

    Settings settings;
    
    if (!file_exists(settings_file_path))
    {
        settings.launch_on_startup = "1";
        settings.power_saving_mode = "0";
        settings.check_for_updates = "1";
        settings.touch_control = "1";
        settings.table_mode = "0";
        settings.detect_interaction_plane = "0";
        settings.click_height = "5";

        if (!directory_exists(data_path))
            create_directory(data_path);
        
        ofstream settings_ofs(settings_file_path, ios::binary);
        settings_ofs.write((char*)&settings, sizeof(settings));

        console_log("settings file created");
    }
    else
    {
        ifstream ifs(settings_file_path, ios::binary);
        ifs.read((char*)&settings, sizeof(settings));

        console_log("settings file loaded");
    }

    IPC* ipc_ptr = &ipc;
    Settings* settings_ptr = &settings;
    ipc.map_function("get toggles", [ipc_ptr, settings_ptr](const string message_body)
    {
        string response = "";
        response += settings_ptr->launch_on_startup
                 +  settings_ptr->power_saving_mode
                 +  settings_ptr->check_for_updates
                 +  settings_ptr->touch_control
                 +  settings_ptr->table_mode
                 +  settings_ptr->detect_interaction_plane
                 +  settings_ptr->click_height;

        ipc_ptr->send_message("menu_plus", "get toggles", response);
    });

    ipc.map_function("set toggle", [ipc_ptr, settings_ptr](const string message_body)
    {
        const string toggle_name = message_body.substr(0, message_body.size() - 1);
        const string toggle_value = message_body.substr(message_body.size() - 1, message_body.size());

        if (toggle_name == "launch_on_startup")
            settings_ptr->launch_on_startup = toggle_value;

        else if (toggle_name == "power_saving_mode")
            settings_ptr->power_saving_mode = toggle_value;

        else if (toggle_name == "check_for_updates")
            settings_ptr->check_for_updates = toggle_value;

        else if (toggle_name == "touch_control")
            settings_ptr->touch_control = toggle_value;

        else if (toggle_name == "table_mode")
            settings_ptr->table_mode = toggle_value;

        else if (toggle_name == "detect_interaction_plane")
            settings_ptr->detect_interaction_plane = toggle_value;

//.........这里部分代码省略.........
```

开发者ID:bstramsek，项目名称:touch_plus_source_code，代码行数:101，代码来源:[main.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fbstramsek%2Ftouch_plus_source_code)



## 示例13: ToString

▲ 点赞 1 ▼

```cpp
bool EditorMode::PlayProject(String addArgs, bool debug)
{
    FileSystem* fileSystem = GetSubsystem<FileSystem>();
    ToolSystem* tsystem = GetSubsystem<ToolSystem>();
    Project* project = tsystem->GetProject();

    if (!project)
        return false;

    ToolEnvironment* env = GetSubsystem<ToolEnvironment>();

    String playerBinary = env->GetEditorBinary();

    // TODO: We need to configure project as managed
    bool managed = false;
    if (fileSystem->FileExists(project->GetResourcePath() + "AtomicProject.dll"))
    {
        managed = true;
        playerBinary = env->GetAtomicNETManagedIPCPlayerBinary();
    }


    Vector<String> paths;
    paths.Push(env->GetCoreDataDir());
    paths.Push(env->GetPlayerDataDir());
    paths.Push(project->GetResourcePath());

    // fixme: this is for loading from cache
    paths.Push(project->GetProjectPath());
    paths.Push(project->GetProjectPath() + "Cache");

    String resourcePaths;
    resourcePaths.Join(paths, "!");

    Vector<String> vargs;

    String args = ToString("--player --project \"%s\"", AddTrailingSlash(project->GetProjectPath()).CString());

    vargs = args.Split(' ');

    if (managed)
    {            
        vargs.Insert(0, ToString("\"%s\"", (fileSystem->GetProgramDir() + "Resources/").CString()));        
        vargs.Insert(0, "--resourcePrefix");
    }

    if (debug)
        vargs.Insert(0, "--debug");

    if (addArgs.Length() > 0)
        vargs.Insert(0, addArgs.Split(' '));

    String dump;
    dump.Join(vargs, " ");
    ATOMIC_LOGINFOF("Launching Broker %s %s", playerBinary.CString(), dump.CString());

    IPC* ipc = GetSubsystem<IPC>();
    playerBroker_ = ipc->SpawnWorker(playerBinary, vargs);

    if (playerBroker_)
    {
        SubscribeToEvent(playerBroker_, E_IPCWORKERSTART, ATOMIC_HANDLER(EditorMode, HandleIPCWorkerStarted));

        SubscribeToEvent(E_IPCPLAYERPAUSERESUMEREQUEST, ATOMIC_HANDLER(EditorMode, HandleIPCPlayerPauseResumeRequest));
        SubscribeToEvent(E_IPCPLAYERUPDATESPAUSEDRESUMED, ATOMIC_HANDLER(EditorMode, HandleIPCPlayerUpdatesPausedResumed));
        SubscribeToEvent(E_IPCPLAYERPAUSESTEPREQUEST, ATOMIC_HANDLER(EditorMode, HandleIPCPlayerPauseStepRequest));
        SubscribeToEvent(E_IPCPLAYEREXITREQUEST, ATOMIC_HANDLER(EditorMode, HandleIPCPlayerExitRequest));
    

        SubscribeToEvent(playerBroker_, E_IPCJSERROR, ATOMIC_HANDLER(EditorMode, HandleIPCJSError));
        SubscribeToEvent(playerBroker_, E_IPCWORKEREXIT, ATOMIC_HANDLER(EditorMode, HandleIPCWorkerExit));
        SubscribeToEvent(playerBroker_, E_IPCWORKERLOG, ATOMIC_HANDLER(EditorMode, HandleIPCWorkerLog));
    }

    return playerBroker_.NotNull();

}
```

开发者ID:EternalXY，项目名称:AtomicGameEngine，代码行数:77，代码来源:[AEEditorMode.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FEternalXY%2FAtomicGameEngine)



## 示例14: get_Running

▲ 点赞 1 ▼

```cpp
STDMETHODIMP CSimControl::get_Running(BOOL *pVal)
{
    IPC ipc;
    *pVal = ipc.isInit();
    return S_OK;
}
```

开发者ID:BackupTheBerlios，项目名称:sim-im-svn，代码行数:6，代码来源:[SimControl.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FBackupTheBerlios%2Fsim-im-svn)



## 示例15: ipc_thread_function

▲ 点赞 0 ▼

```cpp
void ipc_thread_function()
{
    while (true)
    {
        ipc.update();
        Sleep(100);
    }
}
```

开发者ID:bstramsek，项目名称:touch_plus_source_code，代码行数:8，代码来源:[main.cpp](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fbstramsek%2Ftouch_plus_source_code)
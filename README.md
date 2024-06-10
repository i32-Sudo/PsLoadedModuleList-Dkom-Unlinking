# PsLoadedModuleList-Dkom-Unlinking
PsLoadedModuleList Unlinking through DKOM Manipulation


```cpp

EXTERN_C
PLIST_ENTRY PsLoadedModuleList;

typedef struct _KLDR_DATA_TABLE_ENTRY
{
	LIST_ENTRY InLoadOrderLinks;
	PVOID ExceptionTable;
	ULONG ExceptionTableSize;
	// ULONG padding on IA64
	PVOID GpValue;
	/*PNON_PAGED_DEBUG_INFO*/ PVOID NonPagedDebugInfo;
	PVOID DllBase;
	PVOID EntryPoint;
	ULONG SizeOfImage;
	UNICODE_STRING FullDllName;
	UNICODE_STRING BaseDllName;
	ULONG Flags;
	USHORT LoadCount;
	USHORT __Unused5;
	PVOID SectionPointer;
	ULONG CheckSum;
	// ULONG padding on IA64
	PVOID LoadedImports;
	PVOID PatchInformation;
} KLDR_DATA_TABLE_ENTRY, * PKLDR_DATA_TABLE_ENTRY;

extern "C" void DkomUnlinking() {
	PKLDR_DATA_TABLE_ENTRY pSelfEntry = nullptr;
	auto pNext = PsLoadedModuleList->Flink;
	if (pNext != NULL)
	{
		while (pNext != PsLoadedModuleList)
		{
			auto pEntry = CONTAINING_RECORD(pNext, KLDR_DATA_TABLE_ENTRY, InLoadOrderLinks);

			auto pBase = pEntry->DllBase;
			if (DriverObject->DriverStart == pBase)
			{
				pSelfEntry = pEntry;
				break;
			}

			pNext = pNext->Flink;
		}
	}

	//////////////////////////////////////////////////////////////////////////////////////////////////////////

	if (pSelfEntry)
	{
		KIRQL kIrql = KeRaiseIrqlToDpcLevel();
		auto pPrevEntry = (PKLDR_DATA_TABLE_ENTRY)pSelfEntry->InLoadOrderLinks.Blink;
		auto pNextEntry = (PKLDR_DATA_TABLE_ENTRY)pSelfEntry->InLoadOrderLinks.Flink;
		if (pPrevEntry)
		{
			pPrevEntry->InLoadOrderLinks.Flink = pSelfEntry->InLoadOrderLinks.Flink;
		}
		if (pNextEntry)
		{
			pNextEntry->InLoadOrderLinks.Blink = pSelfEntry->InLoadOrderLinks.Blink;
		}
		pSelfEntry->InLoadOrderLinks.Flink = (PLIST_ENTRY)pSelfEntry;
		pSelfEntry->InLoadOrderLinks.Blink = (PLIST_ENTRY)pSelfEntry;
		KeLowerIrql(kIrql);
	}

	//////////////////////////////////////////////////////////////////////////////////////////////////////////
}
```

# Contact
If you want an actually good Kernel Level Cheat that is UD My discord is -> `_ambitza`

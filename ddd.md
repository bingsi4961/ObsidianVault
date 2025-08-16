```csharp
// 在 Bom90ReportGenerator.cs 中新增以下內容

/// <summary>
/// 產生 KP彙整表 Sheet
/// </summary>
private void GenerateKpSummarySheet(XLWorkbook workBook)
{
    var workSheet = workBook.Worksheets.Add("KP彙整");
    var summaryGenerator = new KpSummarySheetGenerator();
    summaryGenerator.Generate(workSheet, _bom90CompsList);
}

/// <summary>
/// KP彙整表 Sheet 產生器
/// </summary>
public class KpSummarySheetGenerator
{
    private readonly KpSummaryHeaderConfig _headerConfig;

    public KpSummarySheetGenerator()
    {
        _headerConfig = new KpSummaryHeaderConfig();
    }

    /// <summary>
    /// 產生 KP彙整表 內容
    /// </summary>
    public void Generate(IXLWorksheet workSheet, List<Bom90ComponentsModel> bom90CompsList)
    {
        // 建立標題列
        CreateHeaderRow(workSheet);

        // 從所有 90料號 中提取並彙整零件資料
        var kpSummaryData = ExtractAndGroupKpData(bom90CompsList);

        // 產生資料列
        GenerateDataRows(workSheet, kpSummaryData);

        // 設定欄寬和樣式
        SetColumnWidthAndStyle(workSheet);
    }

    /// <summary>
    /// 建立標題列
    /// </summary>
    private void CreateHeaderRow(IXLWorksheet workSheet)
    {
        var allHeaders = _headerConfig.GetAllHeaders();
        var headerColors = _headerConfig.GetHeaderColors();

        for (int i = 0; i < allHeaders.Length; i++)
        {
            var cell = workSheet.Cell(1, i + 1);
            cell.Value = allHeaders[i];
            cell.Style.Font.Bold = true;
            cell.Style.Fill.BackgroundColor = headerColors[i];
            cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
            cell.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
            cell.Style.Font.FontColor = XLColor.White;
        }
    }

    /// <summary>
    /// 從所有 90料號 中提取並彙整零件資料
    /// </summary>
    private KpSummaryMatrix ExtractAndGroupKpData(List<Bom90ComponentsModel> bom90CompsList)
    {
        var matrix = new KpSummaryMatrix();

        // 收集所有零件並按類型分組
        foreach (var bom90Comps in bom90CompsList)
        {
            matrix.AddCpuData(bom90Comps.CPUs, bom90Comps.Part90Quantity);
            matrix.AddPchData(bom90Comps.PCHs, bom90Comps.Part90Quantity);
            matrix.AddGpuData(bom90Comps.GPUs, bom90Comps.Part90Quantity);
            matrix.AddVramData(bom90Comps.VRAMs, bom90Comps.Part90Quantity);
            matrix.AddDimmData(bom90Comps.DIMMs, bom90Comps.Part90Quantity);
            matrix.AddSsdData(bom90Comps.SSDs, bom90Comps.Part90Quantity);
            matrix.AddDdrData(bom90Comps.DDRs, bom90Comps.Part90Quantity);
            matrix.AddHddData(bom90Comps.HDDs, bom90Comps.Part90Quantity);
            matrix.AddWlanData(bom90Comps.WLANs, bom90Comps.Part90Quantity);
            matrix.AddBatteryData(bom90Comps.Batterys, bom90Comps.Part90Quantity);
            matrix.AddVgaData(bom90Comps.VGAs, bom90Comps.Part90Quantity);
        }

        // 進行分組和排序
        matrix.GroupAndSort();

        return matrix;
    }

    /// <summary>
    /// 產生資料列
    /// </summary>
    private void GenerateDataRows(IXLWorksheet workSheet, KpSummaryMatrix matrix)
    {
        // 計算總列數 = 所有零件類型中最大的 distinct 數量
        int totalRows = matrix.GetMaxRowCount();

        for (int rowIndex = 0; rowIndex < totalRows; rowIndex++)
        {
            var currentRow = rowIndex + 2; // 從第2列開始（第1列是標題）
            FillSingleDataRow(workSheet, currentRow, matrix, rowIndex);
        }
    }

    /// <summary>
    /// 填入單列資料
    /// </summary>
    private void FillSingleDataRow(IXLWorksheet workSheet, int rowNum, KpSummaryMatrix matrix, int rowIndex)
    {
        int col = 1;

        // CPU
        if (rowIndex < matrix.CpuSummaries.Count)
        {
            var cpu = matrix.CpuSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(cpu.Specification, cpu.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = cpu.TotalQuantity;
        }
        col += _headerConfig.CpuHeaders.Length;

        // PCH
        if (rowIndex < matrix.PchSummaries.Count)
        {
            var pch = matrix.PchSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(pch.Specification, pch.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = pch.TotalQuantity;
        }
        col += _headerConfig.PchHeaders.Length;

        // GPU
        if (rowIndex < matrix.GpuSummaries.Count)
        {
            var gpu = matrix.GpuSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(gpu.Specification, gpu.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = gpu.TotalQuantity;
        }
        col += _headerConfig.GpuHeaders.Length;

        // VRAM
        if (rowIndex < matrix.VramSummaries.Count)
        {
            var vram = matrix.VramSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(vram.Specification, vram.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = vram.TotalQuantity;
        }
        col += _headerConfig.VramHeaders.Length;

        // On-Board DIMM
        if (rowIndex < matrix.DimmSummaries.Count)
        {
            var dimm = matrix.DimmSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(dimm.Specification, dimm.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = dimm.TotalQuantity;
        }
        col += _headerConfig.DimmHeaders.Length;

        // SSD
        if (rowIndex < matrix.SsdSummaries.Count)
        {
            var ssd = matrix.SsdSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(ssd.Capacity, ssd.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = ssd.TotalQuantity;
        }
        col += _headerConfig.SsdHeaders.Length;

        // DDR
        if (rowIndex < matrix.DdrSummaries.Count)
        {
            var ddr = matrix.DdrSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(ddr.Capacity, ddr.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = ddr.TotalQuantity;
        }
        col += _headerConfig.DdrHeaders.Length;

        // HDD
        if (rowIndex < matrix.HddSummaries.Count)
        {
            var hdd = matrix.HddSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(hdd.Capacity, hdd.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = hdd.TotalQuantity;
        }
        col += _headerConfig.HddHeaders.Length;

        // WLAN
        if (rowIndex < matrix.WlanSummaries.Count)
        {
            var wlan = matrix.WlanSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(wlan.PartNumber, wlan.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = wlan.TotalQuantity;
        }
        col += _headerConfig.WlanHeaders.Length;

        // Battery
        if (rowIndex < matrix.BatterySummaries.Count)
        {
            var battery = matrix.BatterySummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(battery.PartNumber, battery.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = battery.Vendor;
            workSheet.Cell(rowNum, col + 2).Value = battery.Specification;
            workSheet.Cell(rowNum, col + 3).Value = battery.TotalQuantity;
        }
        col += _headerConfig.BatteryHeaders.Length;

        // VGA
        if (rowIndex < matrix.VgaSummaries.Count)
        {
            var vga = matrix.VgaSummaries[rowIndex];
            workSheet.Cell(rowNum, col).Value = FormatWithSubIndicator(vga.PartNumber, vga.IsSub);
            workSheet.Cell(rowNum, col + 1).Value = vga.Specification;
            workSheet.Cell(rowNum, col + 2).Value = vga.TotalQuantity;
        }
    }

    /// <summary>
    /// 格式化替代料標示（加上替代料標示）
    /// </summary>
    private string FormatWithSubIndicator(string value, bool isSub)
    {
        if (string.IsNullOrWhiteSpace(value))
            return string.Empty;

        return isSub ? $"{value} (替代料)" : value;
    }

    /// <summary>
    /// 設定欄寬和樣式
    /// </summary>
    private void SetColumnWidthAndStyle(IXLWorksheet workSheet)
    {
        // 自動調整欄寬
        workSheet.ColumnsUsed().AdjustToContents();
    }
}

/// <summary>
/// KP彙整表 標題分組設定 (名稱、背景顏色)
/// </summary>
public class KpSummaryHeaderConfig
{
    public string[] CpuHeaders { get; } = { "CPU 規格", "Qty" };
    public string[] PchHeaders { get; } = { "PCH 規格", "Qty" };
    public string[] GpuHeaders { get; } = { "GPU 規格", "Qty" };
    public string[] VramHeaders { get; } = { "VRAM 規格", "Qty" };
    public string[] DimmHeaders { get; } = { "On-Board DIMM 規格", "Qty" };
    public string[] SsdHeaders { get; } = { "SSD 容量", "Qty" };
    public string[] DdrHeaders { get; } = { "DDR 容量", "Qty" };
    public string[] HddHeaders { get; } = { "HDD 容量", "Qty" };
    public string[] WlanHeaders { get; } = { "WLAN 料號", "Qty" };
    public string[] BatteryHeaders { get; } = { "Battery 料號", "Battery 廠商", "Battery 規格", "Qty" };
    public string[] VgaHeaders { get; } = { "VGA 料號", "VGA 規格", "Qty" };

    /// <summary>
    /// 取得 KP彙整表 所有標題
    /// </summary>
    public string[] GetAllHeaders()
    {
        var allHeaders = new List<string>();

        allHeaders.AddRange(CpuHeaders);
        allHeaders.AddRange(PchHeaders);
        allHeaders.AddRange(GpuHeaders);
        allHeaders.AddRange(VramHeaders);
        allHeaders.AddRange(DimmHeaders);
        allHeaders.AddRange(SsdHeaders);
        allHeaders.AddRange(DdrHeaders);
        allHeaders.AddRange(HddHeaders);
        allHeaders.AddRange(WlanHeaders);
        allHeaders.AddRange(BatteryHeaders);
        allHeaders.AddRange(VgaHeaders);

        return allHeaders.ToArray();
    }

    /// <summary>
    /// 取得 KP彙整表 標題顏色
    /// </summary>
    public XLColor[] GetHeaderColors()
    {
        var colors = new List<XLColor>();

        // CPU - 橙色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(243, 156, 18), CpuHeaders.Length));

        // PCH - 紫色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(155, 89, 182), PchHeaders.Length));

        // GPU - 青綠色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(26, 188, 156), GpuHeaders.Length));

        // VRAM - 深灰色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(52, 73, 94), VramHeaders.Length));

        // DIMM - 青色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(22, 160, 133), DimmHeaders.Length));

        // SSD - 綠色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(39, 174, 96), SsdHeaders.Length));

        // DDR - 藍色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(41, 128, 185), DdrHeaders.Length));

        // HDD - 紫藍色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(142, 68, 173), HddHeaders.Length));

        // WLAN - 橘紅色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(211, 84, 0), WlanHeaders.Length));

        // Battery - 深紅色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(192, 57, 43), BatteryHeaders.Length));

        // VGA - 灰色
        colors.AddRange(Enumerable.Repeat(XLColor.FromArgb(127, 140, 141), VgaHeaders.Length));

        return colors.ToArray();
    }
}

/// <summary>
/// KP彙整表資料矩陣
/// </summary>
public class KpSummaryMatrix
{
    public List<KpCpuSummary> CpuSummaries { get; private set; } = new List<KpCpuSummary>();
    public List<KpPchSummary> PchSummaries { get; private set; } = new List<KpPchSummary>();
    public List<KpGpuSummary> GpuSummaries { get; private set; } = new List<KpGpuSummary>();
    public List<KpVramSummary> VramSummaries { get; private set; } = new List<KpVramSummary>();
    public List<KpDimmSummary> DimmSummaries { get; private set; } = new List<KpDimmSummary>();
    public List<KpSsdSummary> SsdSummaries { get; private set; } = new List<KpSsdSummary>();
    public List<KpDdrSummary> DdrSummaries { get; private set; } = new List<KpDdrSummary>();
    public List<KpHddSummary> HddSummaries { get; private set; } = new List<KpHddSummary>();
    public List<KpWlanSummary> WlanSummaries { get; private set; } = new List<KpWlanSummary>();
    public List<KpBatterySummary> BatterySummaries { get; private set; } = new List<KpBatterySummary>();
    public List<KpVgaSummary> VgaSummaries { get; private set; } = new List<KpVgaSummary>();

    // 臨時收集用的字典
    private Dictionary<string, KpCpuSummary> _cpuDict = new Dictionary<string, KpCpuSummary>();
    private Dictionary<string, KpPchSummary> _pchDict = new Dictionary<string, KpPchSummary>();
    private Dictionary<string, KpGpuSummary> _gpuDict = new Dictionary<string, KpGpuSummary>();
    private Dictionary<string, KpVramSummary> _vramDict = new Dictionary<string, KpVramSummary>();
    private Dictionary<string, KpDimmSummary> _dimmDict = new Dictionary<string, KpDimmSummary>();
    private Dictionary<string, KpSsdSummary> _ssdDict = new Dictionary<string, KpSsdSummary>();
    private Dictionary<string, KpDdrSummary> _ddrDict = new Dictionary<string, KpDdrSummary>();
    private Dictionary<string, KpHddSummary> _hddDict = new Dictionary<string, KpHddSummary>();
    private Dictionary<string, KpWlanSummary> _wlanDict = new Dictionary<string, KpWlanSummary>();
    private Dictionary<string, KpBatterySummary> _batteryDict = new Dictionary<string, KpBatterySummary>();
    private Dictionary<string, KpVgaSummary> _vgaDict = new Dictionary<string, KpVgaSummary>();

    public void AddCpuData(List<CpuModel> cpus, int part90Quantity)
    {
        foreach (var cpu in cpus)
        {
            var key = $"{cpu.Specification}|{cpu.IS_SUB}";
            if (!_cpuDict.ContainsKey(key))
            {
                _cpuDict[key] = new KpCpuSummary
                {
                    Specification = cpu.Specification,
                    IsSub = cpu.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _cpuDict[key].TotalQuantity += part90Quantity * (cpu.Quantity ?? 0);
        }
    }

    public void AddPchData(List<PchModel> pchs, int part90Quantity)
    {
        foreach (var pch in pchs)
        {
            var key = $"{pch.Specification}|{pch.IS_SUB}";
            if (!_pchDict.ContainsKey(key))
            {
                _pchDict[key] = new KpPchSummary
                {
                    Specification = pch.Specification,
                    IsSub = pch.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _pchDict[key].TotalQuantity += part90Quantity * (pch.Quantity ?? 0);
        }
    }

    public void AddGpuData(List<GpuModel> gpus, int part90Quantity)
    {
        foreach (var gpu in gpus)
        {
            var key = $"{gpu.Specification}|{gpu.IS_SUB}";
            if (!_gpuDict.ContainsKey(key))
            {
                _gpuDict[key] = new KpGpuSummary
                {
                    Specification = gpu.Specification,
                    IsSub = gpu.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _gpuDict[key].TotalQuantity += part90Quantity * (gpu.Quantity ?? 0);
        }
    }

    public void AddVramData(List<VramModel> vrams, int part90Quantity)
    {
        foreach (var vram in vrams)
        {
            var key = $"{vram.Specification}|{vram.IS_SUB}";
            if (!_vramDict.ContainsKey(key))
            {
                _vramDict[key] = new KpVramSummary
                {
                    Specification = vram.Specification,
                    IsSub = vram.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _vramDict[key].TotalQuantity += part90Quantity * (vram.Quantity ?? 0);
        }
    }

    public void AddDimmData(List<DimmModel> dimms, int part90Quantity)
    {
        foreach (var dimm in dimms)
        {
            var key = $"{dimm.Specification}|{dimm.IS_SUB}";
            if (!_dimmDict.ContainsKey(key))
            {
                _dimmDict[key] = new KpDimmSummary
                {
                    Specification = dimm.Specification,
                    IsSub = dimm.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _dimmDict[key].TotalQuantity += part90Quantity * (dimm.Quantity ?? 0);
        }
    }

    public void AddSsdData(List<SsdModel> ssds, int part90Quantity)
    {
        foreach (var ssd in ssds)
        {
            var key = $"{ssd.Capacity}|{ssd.IS_SUB}";
            if (!_ssdDict.ContainsKey(key))
            {
                _ssdDict[key] = new KpSsdSummary
                {
                    Capacity = ssd.Capacity,
                    IsSub = ssd.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _ssdDict[key].TotalQuantity += part90Quantity * (ssd.Quantity ?? 0);
        }
    }

    public void AddDdrData(List<DdrModel> ddrs, int part90Quantity)
    {
        foreach (var ddr in ddrs)
        {
            var key = $"{ddr.Capacity}|{ddr.IS_SUB}";
            if (!_ddrDict.ContainsKey(key))
            {
                _ddrDict[key] = new KpDdrSummary
                {
                    Capacity = ddr.Capacity,
                    IsSub = ddr.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _ddrDict[key].TotalQuantity += part90Quantity * (ddr.Quantity ?? 0);
        }
    }

    public void AddHddData(List<HddModel> hdds, int part90Quantity)
    {
        foreach (var hdd in hdds)
        {
            var key = $"{hdd.Capacity}|{hdd.IS_SUB}";
            if (!_hddDict.ContainsKey(key))
            {
                _hddDict[key] = new KpHddSummary
                {
                    Capacity = hdd.Capacity,
                    IsSub = hdd.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _hddDict[key].TotalQuantity += part90Quantity * (hdd.Quantity ?? 0);
        }
    }

    public void AddWlanData(List<WlanModel> wlans, int part90Quantity)
    {
        foreach (var wlan in wlans)
        {
            var key = $"{wlan.PartNumber}|{wlan.IS_SUB}";
            if (!_wlanDict.ContainsKey(key))
            {
                _wlanDict[key] = new KpWlanSummary
                {
                    PartNumber = wlan.PartNumber,
                    IsSub = wlan.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _wlanDict[key].TotalQuantity += part90Quantity * (wlan.Quantity ?? 0);
        }
    }

    public void AddBatteryData(List<BatteryModel> batterys, int part90Quantity)
    {
        foreach (var battery in batterys)
        {
            var key = $"{battery.PartNumber}|{battery.Vendor}|{battery.Specification}|{battery.IS_SUB}";
            if (!_batteryDict.ContainsKey(key))
            {
                _batteryDict[key] = new KpBatterySummary
                {
                    PartNumber = battery.PartNumber,
                    Vendor = battery.Vendor,
                    Specification = battery.Specification,
                    IsSub = battery.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _batteryDict[key].TotalQuantity += part90Quantity * (battery.Quantity ?? 0);
        }
    }

    public void AddVgaData(List<VgaModel> vgas, int part90Quantity)
    {
        foreach (var vga in vgas)
        {
            var key = $"{vga.PartNumber}|{vga.Specification}|{vga.IS_SUB}";
            if (!_vgaDict.ContainsKey(key))
            {
                _vgaDict[key] = new KpVgaSummary
                {
                    PartNumber = vga.PartNumber,
                    Specification = vga.Specification,
                    IsSub = vga.IS_SUB,
                    TotalQuantity = 0
                };
            }
            _vgaDict[key].TotalQuantity += part90Quantity * (vga.Quantity ?? 0);
        }
    }

    /// <summary>
    /// 進行分組和排序
    /// </summary>
    public void GroupAndSort()
    {
        // 排序邏輯：IS_SUB(asc) → 其他欄位(asc)
        CpuSummaries = _cpuDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Specification).ToList();
        PchSummaries = _pchDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Specification).ToList();
        GpuSummaries = _gpuDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Specification).ToList();
        VramSummaries = _vramDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Specification).ToList();
        DimmSummaries = _dimmDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Specification).ToList();
        SsdSummaries = _ssdDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Capacity).ToList();
        DdrSummaries = _ddrDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Capacity).ToList();
        HddSummaries = _hddDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.Capacity).ToList();
        WlanSummaries = _wlanDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.PartNumber).ToList();
        BatterySummaries = _batteryDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.PartNumber).ThenBy(x => x.Vendor).ThenBy(x => x.Specification).ToList();
        VgaSummaries = _vgaDict.Values.OrderBy(x => x.IsSub).ThenBy(x => x.PartNumber).ThenBy(x => x.Specification).ToList();
    }

    /// <summary>
    /// 取得最大列數
    /// </summary>
    public int GetMaxRowCount()
    {
        return new[] {
            CpuSummaries.Count,
            PchSummaries.Count,
            GpuSummaries.Count,
            VramSummaries.Count,
            DimmSummaries.Count,
            SsdSummaries.Count,
            DdrSummaries.Count,
            HddSummaries.Count,
            WlanSummaries.Count,
            BatterySummaries.Count,
            VgaSummaries.Count
        }.Max();
    }
}

#region KP彙整表資料模型

public class KpCpuSummary
{
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpPchSummary
{
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpGpuSummary
{
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpVramSummary
{
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpDimmSummary
{
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpSsdSummary
{
    public string Capacity { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpDdrSummary
{
    public string Capacity { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpHddSummary
{
    public string Capacity { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpWlanSummary
{
    public string PartNumber { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpBatterySummary
{
    public string PartNumber { get; set; }
    public string Vendor { get; set; }
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

public class KpVgaSummary
{
    public string PartNumber { get; set; }
    public string Specification { get; set; }
    public bool IsSub { get; set; }
    public decimal TotalQuantity { get; set; }
}

#endregion
```
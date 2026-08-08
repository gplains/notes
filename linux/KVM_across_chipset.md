---
title:KVM_accross_chipset
---

# KVM移行時に引っかかるチップセット問題


### Geminiが推定する影響範囲

- RHEL7.x以前のすべてのディストリ

  RHEL7.5(Ubuntuもたぶん16.04くらい)にq35対応しているので、最初から7.xはi440fxに置き換える腹積もりが必要

- Windows Server 2019以前

  同様の理由で、Windows Server 2016/2019 もi440fxにした方がいいらしい

  Windows Server 2012以前(まだ残っていたら)はもちろんi440fx世代のチップセットが向いてる

### 具体的な設定変更箇所

- virtio_net
  
  ネットワークアダプタで model type=virtio のところ virtio-transitional に書き換えても一応動く

- virtio_console

  所謂シリアルポート

  model type=virtio-transitional を追記してやるとうまくいくこともある、めんどくさい

  こいつが動かないとqemu-guest-agent(qemu-ga)が中央サーバとやりとりしない

- virtio_blk / virtio_scsi

  そもそもOSが起動しなくて泣けるので面倒な人は sdaの sata みたいに雑に書き換えたほうが楽

- qemu-guest-agent

  特にV2V移行では移行元がKVMとかでないとqemu-gaって入ってないのでまあrpmで事前に取っておくと吉


### conver.py

HPEのHVMで間違ってq35で生成されたXML定義をi440fxに書き換えるやつ

sudo virsh list --all でVM名を拾って
sudo python convert.py <VM名> で実行
念のため作業前後で
sudo virsh dumpxml <VM名>  > /tmp/<VM名>.xml して
バックアップを取っておくと吉

```
#!/usr/bin/env python3
import sys
import xml.etree.ElementTree as ET
import subprocess

# qemu XML名前空間の定義
NS = {'qemu': 'http://libvirt.org/schemas/domain/qemu/1.0'}
ET.register_namespace('qemu', NS['qemu'])

def convert_q35_to_i440fx(vm_name):
    # 1. libvirtからXMLを取得
    cmd = f"sudo virsh dumpxml {vm_name}"
    try:
        xml_data = subprocess.check_output(cmd, shell=True, text=True)
    except subprocess.CalledProcessError:
        print(f"[ERROR] Could not dumpxml for {vm_name}")
        sys.exit(1)

    root = ET.fromstring(xml_data)

    # 2. os/type の machine 属性を変更
    os_type = root.find("./os/type")
    if os_type is not None:
        os_type.attrib["machine"] = "pc-i440fx-noble"

    # 3. <qemu:commandline> 内の machine 引数を置換
    qemu_cmd = root.find("./qemu:commandline", NS)
    if qemu_cmd is not None:
        args = qemu_cmd.findall("./qemu:arg", NS)
        for arg in args:
            val = arg.attrib.get("value", "")
            if "pc-q35" in val:
                arg.attrib["value"] = val.replace("pc-q35-8.2", "pc-i440fx-noble").replace("pc-q35", "pc-i440fx-noble")
                if "smbios-entry-point-type" not in arg.attrib["value"]:
                    arg.attrib["value"] += ",smbios-entry-point-type=32"

    # 4. NUMA セクションの削除 ＆ vcpu タグの単純化 (i440fx 互換性のためのクリーンアップ)
    cpu_elem = root.find("./cpu")
    if cpu_elem is not None:
        numa_elem = cpu_elem.find("numa")
        if numa_elem is not None:
            cpu_elem.remove(numa_elem)

    vcpu_elem = root.find("./vcpu")
    if vcpu_elem is not None:
        # current や placement 等の q35 向け動的割り当て属性を削除し、純粋な VCPU 数のみにする
        count = vcpu_elem.text
        vcpu_elem.attrib.clear()
        vcpu_elem.text = count

    devices = root.find("./devices")
    if devices is not None:
        # 5. 既存の PCI コントローラーを削除
        controllers_to_remove = [
            c for c in devices.findall("controller")
            if c.attrib.get("type") == "pci"
        ]
        for c in controllers_to_remove:
            devices.remove(c)

        # 6. i440fx 必須の pci-root (pci.0) を作成
        pci_root = ET.Element("controller", {"type": "pci", "index": "0", "model": "pci-root"})
        devices.append(pci_root)

        # 7. q35固有の Watchdog (itco) を削除
        watchdogs_to_remove = devices.findall("watchdog")
        for w in watchdogs_to_remove:
            devices.remove(w)

        # 8. 各デバイスの固定 PCI アドレスを削除して初期化
        for elem in devices.iter():
            address = elem.find("address")
            if address is not None and address.attrib.get("type") == "pci":
                elem.remove(address)

    # 9. クリーンアップしたXMLを libvirt に再定義
    new_xml = ET.tostring(root, encoding="unicode")
    process = subprocess.Popen(
        ["sudo", "virsh", "define", "/dev/stdin"],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True
    )
    stdout, stderr = process.communicate(input=new_xml)

    if process.returncode == 0:
        print(f"[SUCCESS] {vm_name} completely converted to pc-i440fx-noble (NUMA & PCI cleaned up)!")
    else:
        print(f"[ERROR] Failed to define {vm_name}:\n{stderr}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 convert.py <VM_NAME>")
        sys.exit(1)
    convert_q35_to_i440fx(sys.argv[1])

```

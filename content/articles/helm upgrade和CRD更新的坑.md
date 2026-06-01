+++
title = 'Helm Upgrade和CRD更新的坑'
date = 2026-06-01T22:35:15+08:00
draft = false
+++

已经两次被坑到了，现象很诡异，所以记录一下。

# 现象
helm --dry-run 出来的 manifest 里面有新增的字段，但是 helm upgrade 成功后的 CR 里面这个字段值是 null。

# 排查
首先怀疑 CRD 没更新，于是用 kubectl replace 更新 CRD。

之后再用 helm upgrade，之后 CR 里面这个字段还是 null。

一度怀疑 helm 更新后的 CR 被 operator 改回了旧版本，其实不是这个原因。

kubectl get secret sh.helm.release.v1.vdm3000.v10 -o yaml | yq -r .data.release | base64 -d  | base64 -d | gunzip | jq -r '.manifest' | yq 'select(.kind=="Seaweed") | .spec.volume.concurrentUploadLimitMB'
256

于是真相大白。

# RC
是 helm 计算 3 way merge patch 的时候的问题。

首先 helm upgrade 不会更新 crds 目录下面的 crd。

因为以前没更新 crd 的时候 helm upgrade 的时候生成的 manifest 里面有concurrentUploadLimitMB: 256 了，但是因为 crd 没有这个字段所以 apply 到CR 上就是null。但是 helm 会把这个 manifest 保存起来。
后来再次 helm upgrade，helm 会用前一个 revision 保存的 manifest 和现在render出来的manifest 做个diff 来生成 patch，因为 这两个manifest 里面的 concurrentUploadLimitMB 都是 256，所以patch里面就不包含这个字段。所以就不会更新 cr 了。

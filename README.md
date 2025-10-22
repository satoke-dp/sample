## 人事担当者のVCによるログインとその後の認証フローについて

```mermaid
sequenceDiagram
    participant HR as 人事担当者
    participant IssuerFE as 検証サイト(フロントエンド)
    participant IssuerBE as 検証サイト(バックエンド)
    participant MySov as MySov (Wallet)
    participant VDR as DIDレジストリ/Blockchain

    HR->IssuerFE: 1. ログイン開始
    
    %% VC検証要求 (Challenge Generation)
    IssuerFE->IssuerBE: 2. [GET /auth/vc/challenge] ノンス要求
    Note over IssuerBE: 3. 一意のNonceを生成し、サーバーで一時保存 (リプレイ防止)
    IssuerBE-->IssuerFE: 4. 200 OK (Nonce, VP要求URLを返却)
    IssuerFE->MySov: 5. QRコード表示 / VP提示要求 (Nonceと要求クレームを含む)

    %% 署名と提示 (VP Presentation)
    MySov->MySov: 6. 社員証VCを選択し、Nonceを含めてVPに署名 (Holder秘密鍵)
    Note right of MySov: VPをvp_tokenに格納
    MySov->IssuerFE: 7. ブラウザでRedirect URIにリダイレクト (vp_tokenを搭載)
    IssuerFE->IssuerBE: 8. [POST /auth/vc/exchange] vp_tokenをBackendへ転送

    %% 検証とセッション確立 (Verification & Session Grant)
    Note over IssuerBE: 9. VC/VP検証開始
    IssuerBE->VDR: 9a.  Issuer (企業) の公開鍵を取得
    VDR-->IssuerBE: 9b. 公開鍵を返却
    Note over IssuerBE: 9c. 署名/Nonce/VC内容(役職)・失効ステータスを厳格に検証

    alt 検証成功 & 社員権限あり
        IssuerBE->IssuerBE: 10. ユーザーID/ロールをPayloadに持つJWTを生成
        IssuerBE-->IssuerFE: 11. 200 OK (JWT/セッションクッキーを返却)
        IssuerFE->HR: 12. ログイン完了 (セッション確立)
    else 検証失敗 (署名不正, VC失効, ロール不足)
        IssuerBE-->IssuerFE: 11. 401 Unauthorized / 403 Forbidden
        IssuerFE->HR: 12. ログイン失敗画面表示
    end
```


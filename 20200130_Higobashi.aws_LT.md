# ネタ候補メモ

- RDS Proxy
  - 通常時の挙動
    - RDS Proxy自体への接続数が増えた場合のRDS自体の接続数の変化
    - RDS Proxyを介した場合と直接RDSに繋いだ場合のレイテンシーの違い
    ```ruby
    def start
      # 1万件のデータ登録にProxyのありなしでどれくらい時間差が出るか
      10000.times do |i|
        Test.create(name: "proxy #{i}")
      end
    end

    # Proxy介した場合
    # 開始時間：2020-01-07 07:43:36.628795 UTC
    # 終了時間：2020-01-07 07:46:51.795742 UTC

    # 直接接続した場合
    # 開始時間：2020-01-07 07:57:01.413039 UTC
    # 終了時間：2020-01-07 07:58:09.604841 UTC
    ```
  - フェイルオーバー時の挙動
    - フェイルオーバー対応時の挙動
      - ProxyへのDB接続が維持されるか
      
      ```ruby
      require 'net/http'
      def start
        Rails.logger.debug("start: #{Time.zone.now}")
        error_count = 0
        while true do
          begin
            sleep 0.2
            raise "test error" if Net::HTTP.get_response(URI.parse('http://127.0.0.1:3000/test')).code != '200'
            break if error_count > 0
          rescue => e
            Rails.logger.debug("error: #{e.message}")
            error_count += 1
            retry
          end
        end
        Rails.logger.debug("end: #{Time.zone.now}")
      end
      ```
 
    - 書き込み確認
      - 何秒間書き込みできないか（フェイルオーバー自体の時間との差も
      ```ruby
      def show
        Rails.logger.debug("start: #{Time.zone.now.iso8601(3)}")
        Test.create(name: "#{Time.zone.now.iso8601(3)}")
        head :ok
      rescue
        Rails.logger.debug("error: #{Time.zone.now.iso8601(3)}")
        head :forbidden
      end
      ```
    - 読み込み確認
      - 何秒間読み込みできないか（フェイルオーバー自体の時間との差も
      ```ruby
      def show
        Rails.logger.debug("start: #{Time.zone.now.iso8601(3)}")
        Test.where(id: 1).pluck(:id)
        head :ok
      rescue
        Rails.logger.debug("error: #{Time.zone.now.iso8601(3)}")
        head :forbidden
      end
      ```

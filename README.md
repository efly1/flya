
    def _load_from_yfinance(self, ticker, start_date, end_date):
        """从Yahoo Finance API加载数据"""
        try:
            if not start_date or not end_date:
                # 如果没有提供日期范围，使用最近5年的数据
                end_date = datetime.now().strftime('%Y-%m-%d')
                start_date = (datetime.now() - pd.DateOffset(years=5)).strftime('%Y-%m-%d')
            
            data = yf.download(ticker, start=start_date, end=end_date)
            if data.empty:
                print(f"无法获取 {ticker} 的数据")
                return False
            
            self.data = data
            self.processed_data = data.copy()
            return True
        except Exception as e:
            print(f"从Yahoo Finance获取数据时出错: {e}")
            return False
    
    def clean_data(self, methods=None):
        """
        数据清洗
        参数:
            methods: 清洗方法列表，如 ['drop_na', 'fill_na', 'outliers']
        """
        if self.processed_data is None:
            print("没有可处理的数据")
            return False
        
        if methods is None:
            methods = ['drop_na', 'outliers']
        
        data = self.processed_data.copy()
        
        if 'drop_na' in methods:
            # 删除包含缺失值的行
            data = data.dropna()
        
        if 'fill_na' in methods:
            # 填充缺失值
            for col in data.select_dtypes(include=[np.number]).columns:
                data[col] = data[col].fillna(data[col].median())
        

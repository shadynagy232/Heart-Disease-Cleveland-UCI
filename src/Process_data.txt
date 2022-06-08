import hydra
import pandas as pd
from omegaconf import DictConfig
from sklearn.preprocessing import StandardScaler

def load_data(data_name: str) -> pd.DataFrame:
    return pd.read_csv(data_name)


def drop_na(df: pd.DataFrame) -> pd.DataFrame:
    return df.dropna()


def drop_features(df: pd.DataFrame, keep_columns: list):
    df = df[keep_columns]
    return df


def drop_outliers(df: pd.DataFrame, column_threshold: dict):
    for col, threshold in column_threshold.items():
        df = df[df[col] < threshold]
    return df.reset_index(drop=True)


def drop_columns_and_rows(df: pd.DataFrame, keep_columns: list, remove_outliers_threshold: dict):
    return df.pipe(drop_features, keep_columns=keep_columns).pipe(
        drop_outliers, column_threshold=remove_outliers_threshold
    )


def scale_features(df: pd.DataFrame):
    scaler = StandardScaler()
    return pd.DataFrame(scaler.fit_transform(df), columns=df.columns)


@hydra.main(config_path="../config", config_name="main")
def process_data(config: DictConfig):

    process_config = config.process

    df = load_data(config.raw_data.path)
    df = drop_na(df)
    df = drop_columns_and_rows(
        df,
        process_config.keep_columns,
        process_config.remove_outliers_threshold,
    )
    df = scale_features(df)
    df.to_csv(config.intermediate.path, index=False)


if __name__ == "__main__":
    process_data()
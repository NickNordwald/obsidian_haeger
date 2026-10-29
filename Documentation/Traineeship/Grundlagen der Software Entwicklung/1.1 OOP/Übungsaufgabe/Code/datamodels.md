from sqlalchemy import Column, VARCHAR, Date, Integer, Numeric, TIMESTAMP
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass


class Capacity(Base):
    __tablename__ = "capacity"

    product_id = Column(VARCHAR, primary_key=True)
    calendar_date = Column(Date, primary_key=True)
    booked_units = Column(Integer, nullable=False, default=0)
    bookable_units = Column(Integer, nullable=False)
    optional_units = Column(Integer, nullable=False)


class Price(Base):
    __tablename__ = "prices"

    product_id = Column(VARCHAR, primary_key=True)
    change_date = Column(Date, primary_key=True)
    length_of_stay = Column(VARCHAR, primary_key=True)
    price_sterling = Column(Numeric, nullable=False)


class PriceRecommendation(Base):
    __tablename__ = "price_recommendations"

    product_id = Column(VARCHAR, primary_key=True)
    calendar_date = Column(Date, primary_key=True)
    days_until_travel = Column(Integer, nullable=False)
    recommended_adjustment_pct = Column(Numeric, nullable=False)
    calculated_at = Column(TIMESTAMP, nullable=False)
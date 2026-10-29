from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
 
DATABASE_URL = "postgresql://tourism:tourism@localhost:5432/tourism"
 
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
 